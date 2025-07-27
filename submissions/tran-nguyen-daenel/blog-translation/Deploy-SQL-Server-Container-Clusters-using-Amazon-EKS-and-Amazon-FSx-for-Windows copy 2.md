# Triển khai cụm container SQL Server bằng Amazon EKS và Amazon FSx cho Windows

> **📖 Bài viết gốc**:[Deploy SQL Server Container Clusters using Amazon EKS and Amazon FSx for Windows](https://aws.amazon.com/vi/blogs/modernizing-with-aws/deploy-sql-server-container-clusters-using-amazon-eks-and-amazon-fsx-for-windows/)  
> **✍️ Tác giả**: Tekena Orugbani  
> **📅 Ngày xuất bản**: 25 tháng 6, 2025  
> **🌐 Nguồn**: *Microsoft Workloads on AWS*  
> **👨‍💻 Người dịch**: Trần Nguyễn Daenel — *Thực tập sinh FCJ*  
> **🗓️ Ngày dịch**: 01 tháng 7, 2025  

---

## 📋 Tóm tắt

Bài viết hướng dẫn triển khai SQL Server dạng container trên Amazon EKS với lưu trữ liên tục qua Amazon FSx for Windows File Server. Giải pháp tận dụng Kubernetes StatefulSet để đảm bảo tính sẵn sàng cao và khả năng phục hồi sau lỗi. Lưu trữ Multi-AZ giúp duy trì kết nối dữ liệu trong trường hợp sự cố vùng. Ngoài ra, hướng dẫn còn cung cấp cách giảm thiểu lỗi chuyển đổi dự phòng khi sử dụng máy khách Linux. Giải pháp giúp tiết kiệm chi phí bản quyền Windows tới 38% và phù hợp với môi trường phát triển, thử nghiệm và sản xuất.

**🎯 Đối tượng đọc**:DevOps Engineers, Application Developers, Solutions Architects,Cloud Migration Planners, Windows and SQL Server System Administrators
**📊 Độ khó**: Intermediate  
**🏷️ Tags**: Microsoft , SQL Server , SQL Server trên AWS

---

## Giới thiệu

Bài đăng trên blog này hướng dẫn cách triển khai phiên bản SQL Server có tính khả dụng cao trong vùng chứa bằng cách sử dụng <a href="https://aws.amazon.com/vi/eks/"><u>Amazon Elastic Kubernetes Service (Amazon EKS)</u></a> với bộ lưu trữ liên tục được hỗ trợ bởi <a href="https://aws.amazon.com/vi/fsx/windows/"><u>Amazon FSx for Windows File Server</u></a>.

Việc chạy cơ sở dữ liệu và các khối lượng công việc có trạng thái khác trong container đã tăng trưởng đáng kể qua các năm. Theo <a href="https://www.cncf.io/wp-content/uploads/2025/04/cncf_annual_survey24_031225a.pdf"><u>khảo sát thường niên CNCF 2024</u></a>, 74% tổ chức đang sử dụng container để quản lý các ứng dụng có trạng thái, tăng 10% so với năm 2023. Việc chạy SQL Server trong container mang lại nhiều lợi ích. Bạn có thể nhanh chóng tạo và khởi động nhiều phiên bản SQL Server để phát triển hoặc thử nghiệm, tận dụng quy trình làm việc song song nhanh chóng. Container cũng tối đa hóa mật độ tài nguyên, cho phép nhiều phiên bản chạy hiệu quả trên cùng một máy chủ, lý tưởng cho các kiến trúc vi dịch vụ trong môi trường thử nghiệm hoặc sản xuất. Và khi bạn chạy SQL Server trong container, bạn tiết kiệm tới 38% chi phí bản quyền Windows vì SQL Server chỉ được hỗ trợ trong container Linux.

## Ví dụ về tiết kiệm chi phí

Bên cạnh những lợi ích khác nhau đã nêu ở trên liên quan đến việc chạy SQL Server trong vùng chứa, đây là ví dụ so sánh chi phí hàng năm giữa Linux và Windows với phiên bản <a href="https://aws.amazon.com/vi/ec2/"><u>Amazon Elastic Compute Cloud (Amazon EC2)</u></a> theo yêu cầu, 16 lõi, r6idn.4xlarge .

| Hệ điều hành | Kiểu phiên bản | Chi phí hàng năm       |
|-------------|----------------|------------------------|
| Linux       | r5b.4xlarge     | 10,441.929 đô la       |
| Cửa sổ      | r5b.4xlarge     | 16,889.28 đô la        |

Dựa trên ví dụ này, việc chạy SQL Server trên Linux có thể tiết kiệm hơn 38%. (Số liệu chính xác tính đến ngày 24 tháng 6 năm 2025, tại khu vực us-east-1.)

## Tổng quan về giải pháp

Chúng tôi sử dụng <a href="https://hub.docker.com/r/microsoft/mssql-server"><u>ảnh container SQL Server</u></a> để triển khai SQL Server dưới dạng Kubernetes <a href="https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/"><u>StatefulSet</u></a> trong cụm Amazon EKS, với FSx for Windows File Server cung cấp bộ nhớ lưu trữ liên tục cho các pod SQL Server. Amazon EKS điều phối các tài nguyên trong cụm và cung cấp tính khả dụng cao cho các pod. Nếu một container (phiên bản) SQL Server gặp sự cố, Amazon EKS sẽ tự động khởi động một container SQL Server mới trong một pod mới và kết nối lại FSx for Windows File Server. Nếu một nút gặp sự cố, Amazon EKS sẽ tạo một pod mới trên một nút đang hoạt động và gắn bộ nhớ lưu trữ. Phương pháp được Kubernetes quản lý này đảm bảo cụm luôn linh hoạt trước các sự cố của phiên bản và nút SQL Server, đồng thời duy trì quyền truy cập cơ sở dữ liệu liên tục.

<img width="1025" height="1022" alt="Image" src="https://github.com/user-attachments/assets/7658b657-e5a4-4173-b1b0-ab59f8dbb0fb" />

Hình 1: SQL Server trên Amazon EKS và Amazon FSx cho Windows

## Tùy chọn lưu trữ cho các container SQL Server trên AWS

AWS cung cấp nhiều tùy chọn lưu trữ liên tục khác nhau cho SQL Server được đóng gói trên Amazon EKS. Chúng bao gồm <a href="https://aws.amazon.com/vi/ebs/"><u> Amazon Elastic Block Store (Amazon EBS)</u></a> , <a href="https://aws.amazon.com/vi/fsx/windows/"><u>Amazon FSx for Windows File Server</u></a> và <a href="https://aws.amazon.com/vi/fsx/netapp-ontap/"><u>Amazon FSx for NetApp ONTAP</u></a> .

Amazon EBS cung cấp khả năng lưu trữ liên tục cho các ứng dụng có trạng thái Amazon EKS trong một vùng khả dụng duy nhất. Do đó, các pod được tạo trong các nút trong cùng vùng khả dụng có thể kết nối với các ổ đĩa Amazon EBS để duy trì tính liên tục trong suốt quá trình khởi động lại và thay thế pod. Các tùy chọn lưu trữ liên tục đa vùng sẵn sàng cho khối lượng công việc Amazon EKS sử dụng Amazon EBS có sẵn thông qua các giải pháp của bên thứ ba như <a href="https://aws.amazon.com/vi/blogs/database/running-highly-available-microsoft-sql-server-containers-in-amazon-eks-with-portworx-cloud-native-storage/"><u> Portworx</u></a> và <a href="https://aws.amazon.com/vi/blogs/modernizing-with-aws/containerize-sql-server-workloads-with-amazon-eks-and-dxoperator-for-kubernetes/"><u> DH2i</u></a> .

Lưu trữ bền vững Multi-AZ gốc cho khối lượng công việc SQL Server được chứa trong container của bạn hiện có sẵn trên Amazon FSx for Windows File Server hoặc Amazon FSx for NetApp ONTAP. Các giải pháp lưu trữ này cung cấp khả năng cho các pod Amazon EKS kết nối với lưu trữ bền vững khi được lên lịch trên các nút chạy trong nhiều vùng khả dụng trong VPC của bạn. Để tìm hiểu thêm về việc triển khai Amazon FSx for NetApp ONTAP, hãy đọc bài viết <a href="https://aws.amazon.com/vi/blogs/storage/run-containerized-applications-efficiently-using-amazon-fsx-for-netapp-ontap-and-amazon-eks/"><u> " Chạy ứng dụng chứa trong container hiệu quả bằng Amazon FSx for NetApp ONTAP và Amazon EKS"</u></a>. Chúng tôi sẽ thảo luận về việc tận dụng Amazon FSx for Windows File Server trong bài viết này.

## Những cân nhắc chính

- Đối với cơ sở dữ liệu sản xuất, Microsoft <a href="https://techcommunity.microsoft.com/blog/sqlserver/update--beta-program-for-sql-server-on-windows-container-is-suspended-/2516639/replies/2564521"><u> chỉ hỗ trợ các container SQL Server trên Linux</u></a> và <a href="https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-kubernetes-best-practices-statefulsets?view=sql-server-ver16"><u> khuyến nghị</u></a> triển khai một container phiên bản SQL Server cho mỗi pod trong cụm Kubernetes. Để đạt được điều này, hãy đặt giá trị "bản sao" thành 1 trong tệp kê khai khi triển khai StatefulSet cho máy chủ SQL trên Amazon EKS. Việc sử dụng loại khối lượng công việc Kubernetes <a href="https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/"><u>StatefulSets</u></a> cho SQL Server sẽ đảm bảo tính cố định danh tính. Điều này đảm bảo việc gán một danh tính ổn định, duy nhất và liên tục trên mỗi pod trong StatefulSet, ngay cả khi lên lịch lại, thay thế hoặc mở rộng quy mô các pod trên Amazon EKS.
 
- Khi tạo Multi-AZ, Amazon FSx for Windows File Server, nó cung cấp một File Server ưa thích và một Standby File Server để truy cập hệ thống tệp. Multi-AZ FSx File Server tự động chuyển đổi dự phòng từ máy chủ ưa thích sang máy chủ tệp dự phòng trong các sự kiện dịch vụ vùng, bảo trì theo kế hoạch hoặc nếu máy chủ tệp ưa thích không khả dụng. Tuy nhiên, điều này chỉ áp dụng trên máy khách Windows theo mặc định. Máy khách Linux không hỗ trợ chuyển đổi dự phòng dựa trên DNS tự động và không tự động kết nối với máy chủ tệp dự phòng trong quá trình chuyển đổi dự phòng hoặc tiếp tục hoạt động của hệ thống tệp sau khi hệ thống tệp Multi-AZ trở lại máy chủ tệp ưa thích do lỗi. Do đó, các vùng chứa SQL Server trên Linux sử dụng FSx for Windows File Server sẽ gặp sự cố trong quá trình bảo trì. Là một phần của giải pháp được đề xuất, tôi đã cung cấp một cơ chế để giảm thiểu hạn chế này trong phần hướng dẫn.

## Điều kiện tiên quyết

Các bước trong hướng dẫn này giả định rằng bạn đã:

- Một cụm Amazon EKS. Nếu không, hãy sử dụng hướng dẫn của chúng tôi để <a href="https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html"><u>tạo cụm Amazon EKS.</u></a>.
- Máy chủ tệp Amazon FSx đa vùng (Multi-AZ) dành cho Windows. Nếu không, hãy sử dụng hướng dẫn của chúng tôi để <a href="https://docs.aws.amazon.com/fsx/latest/WindowsGuide/getting-started.html#getting-started-step1"><u>tạo hệ thống tệp của bạn.</u></a>
- Tài khoản người dùng Active Directory có quyền đọc/ghi vào FSx for Windows File Server.
- Đã cài đặt <a href="https://helm.sh/docs/intro/install/"><u>Helm.</u></a> để triển khai <a href="https://github.com/kubernetes-csi/csi-driver-smb"><u>Trình điều khiển SMB CSI.</u></a>
- Giấy phép chạy Microsoft SQL Server trên AWS. Chúng tôi sẽ sử dụng <a href="https://hub.docker.com/r/microsoft/mssql-server"><u>ảnh container SQL Server Developer Edition</u></a>, miễn phí cho mục đích phát triển và thử nghiệm.

## Hướng dẫn

Trong hướng dẫn sau đây, chúng tôi sẽ hướng dẫn từng bước để triển khai phiên bản SQL Server trong cụm Amazon EKS.

1. Cài đặt <a href="https://github.com/kubernetes-csi/csi-driver-smb"><u>Trình điều khiển SMB CSI</u></a> cho phép Kubernetes truy cập vào máy chủ SMB (trong trường hợp này là Amazon FSx for Windows File Server) trên cả nút Linux và Windows.

```bash
   Bash
    helm repo add csi-driver-smb https://raw.githubusercontent.com/kubernetes-csi/csi-driver-smb/master/charts
    helm install csi-driver-smb csi-driver-smb/csi-driver-smb --namespace kube-system --set windows.enabled=true --version v1.17.0
```

2. Tạo một bí mật Kubernetes để lưu trữ thông tin đăng nhập SMB để truy cập FSx. Các pod SQL Server sẽ yêu cầu thông tin đăng nhập hợp lệ để kết nối với Amazon FSx for Windows File Server để đọc và ghi dữ liệu. Trong bước này, chúng ta sẽ tạo một bí mật Kubernetes chứa tên người dùng và mật khẩu Active Directory với các quyền đọc/ghi cần thiết. Tham khảo <a href="https://docs.aws.amazon.com/eks/latest/best-practices/data-encryption-and-secrets-management.html#_secrets_management"><u>tài liệu Amazon EKS</u></a> để tìm hiểu về các phương pháp hay nhất để quản lý bí mật Kubernetes trên AWS.

```bash
Bash
    kubectl create secret generic fsx-creds --from-literal domain=YourADDomain --from-literal username=UserNameForFsx --from-literal password=<YourPasswordGoesHere>
```
3. Tạo mật khẩu cho người dùng sa để truy cập phiên bản SQL Server. Mật khẩu này phải đáp ứng <a href="https://learn.microsoft.com/en-us/sql/relational-databases/security/password-policy?view=sql-server-ver16"><u>chính sách mật khẩu được khuyến nghị.</u></a>

```bash
Bash
    kubectl create secret generic mssql-creds --from-literal=MSSQL_SA_PASSWORD=<YourSAPassword>
```

4. Sử dụng tệp mssql.conf để tạo <a href="https://kubernetes.io/docs/concepts/configuration/configmap/"><u>ConfigMap</u></a> trong Kubernetes. Trước tiên, hãy định nghĩa cấu hình SQL Server của bạn trong tệp mssql.conf.

```Conf
Conf
    [EULA]
    accepteula = Y
    accepteulaml = Y

    [filelocation]
    defaultdatadir = /var/opt/mssql/userdata
    defaultlogdir = /var/opt/mssql/userlog

    [memory]
    memorylimitmb = 28672

    [control]
    WriteThrough=1
    AlternativeWriteThrough=0

    [traceflag]
    traceflag0=3979
```

Thứ hai, sử dụng lệnh kubectl create configmap với cờ –from-file để tạo một ConfigMap từ tệp mssql.conf. Các pod SQL Server sẽ sử dụng ConfigMap này để truy cập các thiết lập cấu hình.

```bash
   Bash
    kubectl create configmap mssqlconf --from-file mssql.conf
```

5. Cung cấp dung lượng lưu trữ. Để cung cấp dung lượng lưu trữ cho các container SQL Server, chúng ta sẽ tạo một <a href="https://docs.aws.amazon.com/eks/latest/userguide/create-storage-class.html"><u>Lớp Lưu trữ (Storage Class)</u></a> trong Amazon EKS. Lớp lưu trữ này xác định cách tự động cung cấp Amazon FSx for Windows File Server khi pod yêu cầu lưu trữ liên tục. Thay thế giá trị "source" bằng tên miền đủ điều kiện của Amazon FSx for Windows File Server trong tệp manifest sau và lưu dưới dạng fsx-smb.yaml.

```yaml
YAML
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
    name: fsx-smb
    provisioner: smb.csi.k8s.io
    parameters:
    source: "//amznfsxa7ko7pfq.company.com/share/sql" # Use the FQDN provided by Amazon FSx for Windows File Server
    csi.storage.k8s.io/provisioner-secret-name: "fsx-creds"
    csi.storage.k8s.io/provisioner-secret-namespace: "default"
    csi.storage.k8s.io/node-stage-secret-name: "fsx-creds"
    csi.storage.k8s.io/node-stage-secret-namespace: "default"
    reclaimPolicy: Retain
    volumeBindingMode: WaitForFirstConsumer
    mountOptions:
    - dir_mode=0777
    - file_mode=0777
    - uid=1001
    - gid=1001
```

Sau đó chạy lệnh này để tạo Lớp lưu trữ:
```bash
Bash
    kubectl apply -f fsx-smb.yaml   
```
6. Triển khai StatefulSet cho SQL Server. Tạo một tệp văn bản mới, thêm mã sau và lưu dưới dạng mssql-server.yaml.

```yaml
YAML
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
    name: mssql
    labels:
    app: mssql
    spec:
    serviceName: "mssql"
    replicas: 1
    selector:
    matchLabels:
    app: mssql
    template:
    metadata:
    labels:
        app: mssql
    spec:
    securityContext:
        fsGroup: 10001
    containers:
    - name: mssql
        command:
        - /bin/bash
        - -c
        - cp /var/opt/config/mssql.conf /var/opt/mssql/mssql.conf && /opt/mssql/bin/sqlservr
        image: mcr.microsoft.com/mssql/server:2022-latest
        resources:
        limits:
        memory: 2Gi
        cpu: '2'
        ports:
        - containerPort: 1433
        env:
        - name: MSSQL_PID
        value: "Developer"
        - name: ACCEPT_EULA
        value: "Y"
        - name: MSSQL_ENABLE_HADR
        value: "1"
        - name: MSSQL_SA_PASSWORD
        valueFrom:
            secretKeyRef:
            name: mssql-creds
            key: MSSQL_SA_PASSWORD
        volumeMounts:
        - name: mssql
        mountPath: "/var/opt/mssql"
        - name: userdata
        mountPath: "/var/opt/mssql/userdata"
        - name: userlog
        mountPath: "/var/opt/mssql/userlog"
        - name: tempdb
        mountPath: "/var/opt/mssql/tempdb"
        - name: mssqlconf
        mountPath: "/var/opt/config"
    volumes:
        - name: mssqlconf
        configMap:
            name: mssqlconf
    volumeClaimTemplates:
    - metadata:
        name: mssql
        spec:
        accessModes:
        - ReadWriteOnce
        resources:
        requests:
            storage: 8Gi
        storageClassName: "fsx-smb"
    - metadata:
        name: userdata
        spec:
        accessModes:
        - ReadWriteOnce
        resources:
        requests:
            storage: 8Gi
        storageClassName: "fsx-smb"
    - metadata:
        name: userlog
        spec:
        accessModes:
        - ReadWriteOnce
        resources:
        requests:
            storage: 8Gi
        storageClassName: "fsx-smb"
    - metadata:
        name: tempdb
        spec:
        accessModes:
        - ReadWriteOnce
        resources:
        requests:
            storage: 8Gi
        storageClassName: "fsx-smb"
    ---
    apiVersion: v1
    kind: Service
    metadata:
    name: mssql-server
    spec:
    selector:
        app: mssql
    ports:
        - protocol: TCP
        port: 1433
        targetPort: 1433
    type: ClusterIP
```

Sau đó chạy lệnh này để tạo StatefulSet:
```bash
Bash
    kubectl apply -f mssql-server.yaml
``` 
7. Xem trạng thái của phiên bản SQL Server của bạn trong cụm Amazon EKS. Sau khi triển khai thành công vùng chứa SQL Server trong cụm Amazon EKS, hãy thực hiện các lệnh sau để xem StatefulSet, pod và điểm cuối dịch vụ SQL Server trong cụm.

```bash
Bash
    kubectl get statefulset
    kubectl get pods
    kubectl get service
```
---
<img width="684" height="294" alt="Image" src="https://github.com/user-attachments/assets/5e1d9635-7543-4697-b381-4a301e243c43" />

Hình 2: Xem StateFulSet, Pod và Service cho SQL Server

Trong ví dụ này, điểm cuối dịch vụ SQL Server có thể truy cập tại 10.100.40.100. Bây giờ chúng ta có thể kết nối với phiên bản SQL Server bằng các công cụ SQL Server tiêu chuẩn như sqlcmd và SSMS.


<img width="690" height="370" alt="Image" src="https://github.com/user-attachments/assets/755b5727-fa44-405b-8364-d79b67d1a32d" />

Hình 3: Kết nối với vùng chứa SQL Server bằng tiện ích sqlcmd


<img width="1335" height="307" alt="Image" src="https://github.com/user-attachments/assets/0b523e5c-df5d-452e-a76a-c98b9ac43ed0" />
Hình 4: Kết nối với vùng chứa SQL Server bằng SSMS

8. Giảm thiểu tình trạng máy khách Linux bị lỗi chuyển đổi dự phòng trên Amazon FSx for Windows File Server. Để giảm thiểu tình trạng máy khách Linux bị lỗi chuyển đổi dự phòng mà chúng ta đã thảo luận trước đó, hãy chạy tập lệnh sau, tập lệnh này sẽ thực hiện các thao tác sau:
   
a. Đặt Thời gian tồn tại (TTL) của các bản ghi DNS của Máy chủ tệp Amazon FSx for Windows thành 30 giây.
b. Giám sát địa chỉ IP của máy chủ tệp Amazon FSx for Windows ưu tiên và máy chủ tệp độc lập.
c. Nếu phát hiện hệ thống tệp không thể truy cập được thông qua máy chủ tệp Amazon FSx for Windows ưu tiên, hệ thống sẽ cập nhật DNS bằng địa chỉ IP dự phòng.
d. Hệ thống tiếp tục giám sát Amazon FSx và cập nhật DNS lại khi hệ thống tệp không thể truy cập trở lại máy chủ tệp ưu tiên.

Đảm bảo thay thế tên máy chủ và địa chỉ IP của Amazon FSx bằng tên máy chủ và địa chỉ IP của Amazon FSx for Windows File Server.

```powerShell
PowerShell
    # Create new folder to store the FSx monitoring script
    New-Item -Path C:\FSxCheck\ -ItemType Directory -Force

    #Set FSx DNS record TTL to 30 seconds
    $dnsServer = ((ipconfig | findstr [0-9].\.)[0]).Split()[-1]
    $domain = (Get-ADDomain).Forest
    $record = Get-DnsServerResourceRecord -ZoneName $Domain -ComputerName $dnsServer | where-Object {$_.HostName -eq "amznfsxa7ko7pfq"} #Replace this with the DNS hostname of your Amazon FSx for Windows File Server

    $newRecord = $record.Clone()
    $newRecord.TimeToLive = [TimeSpan]::FromSeconds(30)

    Set-DnsServerResourceRecord -ComputerName $dnsServer -ZoneName $domain -OldInputObject $record -NewInputObject $newRecord 

    # Script to monitor FSx
    @'
    # Define the file server and port
    #$Server = hostname
    $fileServers = @("11.11.69.37","11.11.98.78") # Replace with your Amazon FSx for Windows File Server IP addresses
    $DCs = (Get-ADDomainController -Filter *).HostName
    $port = 445

    # Function to test the connection
    function Test-Port {
        param (
            [string]$server,
            [int]$port
        )
        
        # Create a TCP client to test the FSx connection
        $tcpClient = New-Object System.Net.Sockets.TcpClient
        
        try {
            # Attempt to connect to the FSx on the specified port
            $tcpClient.Connect($server, $port)
            return $true  # Connection successful
        } catch {
            return $false  # Connection failed
        } finally {
            $tcpClient.Close()
        }
    }

    # Check Fsx endpoint
    foreach ($fileServer in $fileServers){
        if ((Test-Port -server $fileServer -port $port)){
                if (!(Get-DnsServerResourceRecord -ZoneName company.com -ComputerName $DC | Where {$_.RecordData.Ipv4Address -eq $fileServer})){
                    foreach ($DC in $DCs){
                        Add-DnsServerResourceRecord -ZoneName company.com -ComputerName $DC -A -Name "amznfsxa7ko7pfq" -IPv4Address $fileServer -TimeToLive 00:01:00 -AgeRecord
                    }
                }

        }
        else{
            foreach ($DC in $DCs){
                Get-DnsServerResourceRecord -ZoneName company.com | Where {$_.RecordData.Ipv4Address -eq $fileServer} | Remove-DnsServerResourceRecord -ZoneName company.com -Force
            }
        }
    } 
    '@ | Set-Content -Path "C:\FSxCheck\FSxCheck.ps1"

    # Create a new task action
    $taskAction = New-ScheduledTaskAction `
        -Execute 'C:\Windows\System32\cmd.exe' `
        -Argument '/C C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -NoProfile -NonInteractive -NoLogo -ExecutionPolicy Unrestricted -File "C:\FSxCheck\FSxCheck.ps1"'

    # Create a new trigger at startup
    $taskTrigger = New-ScheduledTaskTrigger -Once -At (Get-Date).AddMinutes(1) `
    -RepetitionInterval (New-TimeSpan -Minutes 1) `
    -RepetitionDuration (New-TimeSpan -Days 1827)

    # The name of your scheduled task.
    $taskName = "FSxCheckScript"

    # Describe the scheduled task.
    $description = "Checks that FSx for Window File Server is reachable."

    # Register the scheduled task
    Register-ScheduledTask `
        -TaskName $taskName `
        -Action $taskAction `
        -Trigger $taskTrigger `
        -Description $description `
        -User "System" `
        -RunLevel "Highest"

    Write-Output "Scheduled Task FSxCheckScript created"
```

## Kiểm tra

Để kiểm tra khả năng phục hồi đa vùng sẵn sàng của Amazon FSx for Windows File Server cho việc triển khai SQL Server trên Amazon EKS, bạn có thể tự khởi tạo quá trình chuyển đổi dự phòng của Amazon FSx for Windows File Server. Để thực hiện việc này, <a href="https://docs.aws.amazon.com/fsx/latest/WindowsGuide/increase-throughput-capacity.html"><u>hãy điều chỉnh dung lượng thông lượng</u></a> của Amazon FSx for Windows File Server, điều này sẽ kích hoạt quá trình chuyển đổi dự phòng. Chúng tôi khuyên bạn nên thử nghiệm giải pháp được cung cấp trong bài viết này để xem nó có đáp ứng yêu cầu của bạn hay không trước khi áp dụng vào môi trường sản xuất.

## Dọn dẹp

Để tránh các khoản phí liên tục, hãy xóa mọi tài nguyên bạn đã tạo khi làm theo các bước trong bài đăng trên blog này, bao gồm:

- Cụm Amazon EKS.
- Máy chủ tệp Amazon FSx dành cho Windows.

## Phần kết luận

Việc đóng gói khối lượng công việc cơ sở dữ liệu đã trở nên phổ biến trong những năm gần đây—và có lý do chính đáng. Các tổ chức đang chứng kiến sự gia tăng tính linh hoạt, nhanh nhạy và tiết kiệm chi phí đáng kể. Đặc biệt đáng chú ý là việc tiết kiệm chi phí cho SQL Server. Bạn không chỉ có thể tối đa hóa mật độ tài nguyên trên các máy chủ lưu trữ đóng gói mà còn tiết kiệm chi phí cấp phép Windows. Trong bài viết này, tôi đã chỉ ra cách bạn có thể khai thác những lợi ích này bằng cách triển khai một phiên bản SQL Server có tính khả dụng cao trong một vùng chứa trên Amazon EKS, sử dụng Amazon FSx for Windows File Server để lưu trữ liên tục.

Khám phá <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/optimize-costs-microsoft-workloads/containers-main.html"><u> thêm nội dung </u></a>g để bắt đầu lộ trình hiện đại hóa khối lượng công việc Microsoft của bạn bằng Containers on AWS.

---
AWS sở hữu nhiều dịch vụ và tính năng hơn đáng kể so với bất kỳ nhà cung cấp dịch vụ đám mây nào khác, giúp việc di chuyển các ứng dụng hiện có của bạn lên đám mây và xây dựng gần như mọi thứ bạn có thể tưởng tượng trở nên nhanh chóng, dễ dàng và tiết kiệm chi phí hơn. Hãy trang bị cho các ứng dụng Microsoft của bạn cơ sở hạ tầng cần thiết để thúc đẩy kết quả kinh doanh mong muốn. Truy cập  blog <a href="https://aws.amazon.com/vi/blogs/dotnet/"><u>.NET on AWS</u></a>  và  <a href="https://aws.amazon.com/vi/blogs/database/"><u>AWS Database</u></a> của chúng tôi  để được hướng dẫn thêm và lựa chọn cho khối lượng công việc Microsoft của bạn.  <a href="https://pages.awscloud.com/MAP-windows-contact-us.html"><u>Liên hệ với chúng tôi</u></a>  để bắt đầu hành trình di chuyển và hiện đại hóa ngay hôm nay.


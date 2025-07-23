# Công bố AWS CDK EKS v2 L2 Constructs mới

> **📖 Bài viết gốc**: [Announcing the new AWS CDK EKS v2 L2 Constructs](https://aws.amazon.com/vi/blogs/devops/announcing-the-new-aws-cdk-eks-v2-l2-constructs/)  
> **✍️ Tác giả**: Matteo Luigi Restelli  
> **📅 Ngày xuất bản**: 19 tháng 6, 2025  
> **🌐 Nguồn**: *AWS DevOps & Developer Productivity Blog*  
> **👨‍💻 Người dịch**: Trần Nguyễn Daenel — *Thực tập sinh FCJ*  
> **🗓️ Ngày dịch**: 22 tháng 6, 2025  

---

## 📋 Tóm tắt

Bài viết này giới thiệu về việc tích hợp mô hình AI Claude Sonnet 4 vào Amazon Q Developer CLI, giúp các nhà phát triển nâng cao hiệu quả công việc lập trình, phân tích mã, và sửa lỗi trực tiếp từ dòng lệnh mà không tốn thêm chi phí. Bài viết hướng dẫn cách lựa chọn và chuyển đổi giữa các phiên bản mô hình Claude khác nhau, đồng thời minh họa sức mạnh của Claude Sonnet 4 qua một ví dụ thực tế về việc xây dựng ứng dụng Python, cho thấy khả năng vượt trội so với yêu cầu ban đầu.

**🎯 Đối tượng đọc**: DevOps Engineers, Container Developers, Solutions Architects  
**📊 Độ khó**: Intermediate  
**🏷️ Tags**: AWS Cloud Development Kit, DevOps, EKS

---


## Giới thiệu

Hôm nay, chúng tôi xin công bố phát hành `aws-eks-v2` construct, phiên bản alpha mới của cấu trúc [AWS Cloud Development Kit (CDK)](https://aws.amazon.com/vi/cdk/) L2 dành cho [Amazon Elastic Kubernetes Service (EKS)](https://aws.amazon.com/vi/eks/). Cấu trúc này đại diện cho một thay đổi đáng kể trong cách các nhà phát triển có thể định nghĩa và quản lý môi trường EKS của họ bằng cách sử dụng cơ sở hạ tầng dưới dạng mã. Trong khi vẫn duy trì các khả năng mạnh mẽ của [thư viện tiền nhiệm](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_eks-readme.html) trong việc tạo và quản lý các cụm EKS, bản phát hành alpha này giới thiệu những cải tiến kiến trúc quan trọng giúp tăng cường cả tính linh hoạt và khả năng bảo trì.

AWS Cloud Development Kit (AWS CDK) là một khuôn khổ phát triển phần mềm nguồn mở cho phép bạn xác định cơ sở hạ tầng đám mây của mình bằng các ngôn ngữ lập trình quen thuộc và triển khai thông qua AWS CloudFormation.
CDK sử dụng [các cấu trúc](https://docs.aws.amazon.com/cdk/v2/guide/constructs.html) - một khái niệm trừu tượng hóa theo lớp, trong đó các cấu trúc Lớp 1 (L1) ánh xạ trực tiếp đến các tài nguyên CloudFormation, trong khi các cấu trúc Lớp 2 (L2) cung cấp các API trực quan, các hàm trợ giúp, các mặc định về phương pháp hay nhất và tạo ra nhiều mã mẫu và logic kết dính cho bạn. Phương pháp tiếp cận theo lớp này có nghĩa là bạn có thể di chuyển liền mạch giữa các trừu tượng hóa cấp cao cho các trường hợp sử dụng phổ biến và các định nghĩa tài nguyên cấp thấp khi bạn cần kiểm soát chi tiết. Kết quả là trải nghiệm Cơ sở hạ tầng dưới dạng Mã (IaC) giúp bạn duy trì năng suất đồng thời đảm bảo bạn có quyền truy cập vào toàn bộ sức mạnh của các dịch vụ AWS khi cần.
Bạn có thể đọc thêm về các cấu trúc và lợi ích của chúng trong [hướng dẫn sử dụng CDK](https://docs.aws.amazon.com/cdk/v2/guide/constructs.html).

Trong bài viết này chúng ta sẽ khám phá:

- Lý do đằng sau việc tạo ra một [cấu trúc L2 mới cho EKS](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-eks-v2-alpha-readme.html) và những cải tiến được đưa ra bởi thư viện mới này
- Cách sử dụng cấu trúc EKS v2 mới

## Lý lịch

Amazon EKS là dịch vụ Kubernetes được quản lý, giúp bạn dễ dàng chạy Kubernetes trên AWS mà không cần quản lý mặt phẳng điều khiển hoặc các nút. EKS tự động xử lý các tác vụ quan trọng như vá lỗi, cung cấp nút và nâng cấp. Bạn có thể chạy EKS bằng các phiên bản EC2 cho các nút worker, [AWS Fargate](https://docs.aws.amazon.com/eks/latest/userguide/fargate.html) cho các container không máy chủ hoặc kết hợp cả hai, mang lại sự linh hoạt để lựa chọn tùy chọn tính toán phù hợp cho khối lượng công việc của mình.

Mặc dù [cấu trúc EKS L2 hiện tại](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_eks-readme.html) đã phục vụ khách hàng tốt, chúng tôi đã xác định được các cơ hội để nâng cao hơn nữa trải nghiệm của nhà phát triển và hiệu quả vận hành dựa trên phản hồi của họ. Cấu trúc mới mang lại những cải tiến đáng kể thông qua các tài nguyên [AWS CloudFormation](https://aws.amazon.com/it/cloudformation/) `aws-eks-v2` gốc , [xác thực dựa trên Access Entry](https://docs.aws.amazon.com/eks/latest/userguide/access-entries.html) hiện đại và tính linh hoạt kiến trúc được nâng cao. Các lợi ích chính bao gồm giảm chi phí triển khai, đơn giản hóa quản lý truy cập cụm, hỗ trợ nhiều cụm EKS trong một ngăn xếp duy nhất và kiểm soát chi tiết việc tạo tài nguyên với các tính năng như trình xử lý Lambda tùy chọn. Những cải tiến này giúp khách hàng xây dựng và quản lý cơ sở hạ tầng EKS hiệu quả hơn, đồng thời vẫn duy trì chức năng mạnh mẽ mà họ mong đợi từ các cấu trúc AWS CDK.`kubectl`

## Sử dụng L2

Vì cấu trúc này đang trong giai đoạn alpha, bạn sẽ cần cài đặt và nhập cấu trúc bằng [quy trình thư viện cấu trúc thử nghiệm](https://aws.amazon.com/vi/blogs/developer/experimental-construct-libraries-are-now-available-in-aws-cdk-v2/). Trong giai đoạn alpha, nhóm CDK đang tích cực thu thập phản hồi của khách hàng và lặp lại quá trình triển khai. Khi cấu trúc đáp ứng được tiêu chuẩn về tính khả dụng chung, chúng tôi sẽ tích hợp trực tiếp vào thư viện lõi AWS CDK, giúp nó dễ dàng truy cập như các cấu trúc L1 và L2 khác của chúng tôi. Phương pháp này cho phép chúng tôi nhanh chóng cung cấp các tính năng mới đồng thời đảm bảo chúng đáp ứng các tiêu chuẩn cao mà khách hàng mong đợi.

### Triển khai cụm EKS với cấu hình mặc định

Hãy cùng khám phá cách tạo cụm Amazon EKS bằng `aws-eks-v2 `cấu trúc AWS CDK với yêu cầu cấu hình tối thiểu. Ví dụ sau đây minh họa cách đơn giản nhất để định nghĩa cụm EKS, tận dụng sức mạnh của các giá trị mặc định của CDK.
Việc tạo cụm mới được thực hiện bằng `Cluster` cấu trúc. Thuộc tính bắt buộc duy nhất là phiên bản Kubernetes.

```typescript
Typescript
    import * as eksv2 from '@aws-cdk/aws-eks-v2';

 // Creating an EKS Cluster with default properties
const eksCluster = new eksv2.Cluster(this, 'EksCluster', {
  version: eksv2.KubernetesVersion.V1_32,
});
```
Điều này được thể hiện trong Kiến trúc sau đây như thể hiện trong hình 1:

<p align="center">
  <img width="721" height="701" alt="Image" src="https://github.com/user-attachments/assets/fd341ca6-e3ec-4d85-8daf-565b0d3b8dea" />
</p>
<p align="center"><em>Hình 1 – Cấu trúc CDK L2 v2 cho EKS, Kiến trúc mặc định</em></p>

- <a href="https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html"><u>Amazon Virtual Private Cloud</u></a> (VPC) – Một phân vùng được phân tách logic của AWS Cloud, trải rộng trên hai Vùng Khả dụng, được trang bị Cổng Internet để cho phép giao tiếp an toàn với internet. Thiết kế đa Vùng Khả dụng này giúp đảm bảo các ứng dụng của bạn luôn khả dụng ngay cả khi một Vùng Khả dụng gặp sự cố.

- **Amazon EKS Control Plane** – Một mặt phẳng điều khiển Kubernetes được quản lý hoàn toàn, triển khai trong VPC do AWS quản lý, cung cấp tính khả dụng cao và quản lý phiên bản tự động cho các thành phần mặt phẳng điều khiển Kubernetes.

- **Cơ sở hạ tầng mạng con công cộng – Hai mạng con công cộng, mỗi mạng có một phiên bản** <a href="https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html.html"><u>Cổng NAT</u></a> riêng , cho phép các thành phần cụm của bạn truy cập internet an toàn cho các hoạt động thiết yếu như tải hình ảnh container và tải xuống các bản cập nhật. Các Cổng NAT này cung cấp một đường dẫn ra an toàn, đồng thời bảo vệ khối lượng công việc của bạn khỏi bị tiếp xúc trực tiếp với internet.

- **Cấu hình mạng con riêng** – Hai mạng con riêng được tối ưu hóa để chạy các nút làm việc EKS của bạn, mang lại khả năng bảo mật nâng cao bằng cách cô lập khối lượng công việc của bạn khỏi quyền truy cập internet trực tiếp trong khi vẫn duy trì khả năng giao tiếp với các dịch vụ AWS và internet thông qua Cổng NAT.

- **IAM Security Foundation** – Một bộ <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html"><u>chính sách và vai trò IAM</u></a> toàn diện thực hiện nguyên tắc đặc quyền tối thiểu:

    - Vai trò dịch vụ mặt phẳng điều khiển cho phép EKS quản lý tài nguyên AWS thay mặt bạn
    - Vai trò IAM của nút cho phép các nút công nhân tương tác với các dịch vụ AWS khác và tham gia cụm EKS

Bạn cũng có thể sử dụng `FargateCluster` để cung cấp một cụm chỉ sử dụng công nhân Fargate.

```typescript
Typescript
    import * as eksv2 from '@aws-cdk/aws-eks-v2-alpha';

// Creating an EKS Fargate Cluster with default properties and Fargate workers
    const eksFargateCluster = new eksv2.FargateCluster(this, 'EksFargateCluster', {
        version: eksv2.KubernetesVersion.V1_32,
});
```

Để giúp khách hàng kiểm soát tốt hơn các mẫu truy cập cụm, Trình xử lý Kubectl không được triển khai tự động với cấu hình mặc định. Bạn có thể dễ dàng kích hoạt chức năng này bằng cách cấu hình thuộc `kubectlProviderOptions` khi cần quản lý truy cập kubectl như minh họa bên dưới.

```typescript
Typescript
    import * as eksv2 from '@aws-cdk/aws-eks-v2-alpha';
    import { KubectlV32Layer } from '@aws-cdk/lambda-layer-kubectl-v32';

    // Creating an EKS Cluster with default properties and kubectl handler
    const eksCluster = new eksv2.Cluster(this, 'EksCluster', {
        version: eksv2.KubernetesVersion.V1_32,
        kubectlProviderOptions: {
            kubectlLayer: new KubectlV32Layer(this, 'KubectlLayer'),
        },
    });
```

### Triển khai cụm EKS với chế độ tự động

<a href="https://docs.aws.amazon.com/eks/latest/userguide/automode.html"><u>Chế độ Tự động EKS</u></a> là một bước tiến đáng kể trong cách Amazon EKS quản lý năng lực tính toán cho các cụm Kubernetes. Hệ thống quản lý năng lực thông minh này tự động cung cấp và điều chỉnh quy mô nhóm nút dựa trên nhu cầu khối lượng công việc, loại bỏ nhu cầu lập kế hoạch năng lực thủ công.

Khi bạn tạo cụm mới với `aws-eks-v2`cấu trúc này, Chế độ Tự động EKS được kích hoạt theo mặc định, tức là `DefaultCapacityType.AUTOMODE` được tự động đặt làm loại dung lượng mặc định cho Cụm EKS. Nếu muốn, bạn có thể chỉ định Chế `defaultCapacityType` độ Tự động:

```typescript
Typescript
    import * as eksv2 from '@aws-cdk/aws-eks-v2-alpha';

    // Creating an EKS Cluster with AutoMode
    const eksCluster = new eksv2.Cluster(this, 'EksCluster', {
        version: eksv2.KubernetesVersion.V1_32,
        defaultCapacityType: eksv2.DefaultCapacityType.AUTOMODE, // default value
    });
```

Sau khi triển khai Stack chứa phiên bản xây dựng, trong Bảng điều khiển EKS, bạn sẽ thấy rằng một Cụm EKS đã được tạo với Chế độ tự động được bật:

<p align="center">
  <img width="1622" height="744" alt="Image" src="https://github.com/user-attachments/assets/77780c7e-3f3e-4056-aeb9-615271dbd7b6" />
</p>
<p align="center"><em>Hình 2 – Cụm EKS được triển khai với chế độ tự động</em></p>

Chế độ Tự động nâng cao trải nghiệm Amazon EKS của bạn bằng cách tự động cấu hình hai nhóm nút được thiết kế chiến lược ngay khi cài đặt:

- Nhóm nút hệ thống được tối ưu hóa để chạy các thành phần và tiện ích bổ sung quan trọng của hệ thống cụm, đảm bảo hoạt động cụm đáng tin cậy.
- Một nhóm nút chung được thiết kế riêng cho khối lượng công việc ứng dụng của bạn, mang lại sự linh hoạt cần thiết cho nhiều ứng dụng chứa trong container.

Bạn có thể cấu hình nhóm nút nào sẽ được bật thông qua `compute` thuộc tính:

```typescript
Typescript
    import * as eksv2 from '@aws-cdk/aws-eks-v2-alpha';

// Creating an EKS Cluster with Automode and selecting nodePools
    const eksCluster = new eksv2.Cluster(this, 'EksCluster', {
        version: eksv2.KubernetesVersion.V1_32,
        defaultCapacityType: eksv2.DefaultCapacityType.AUTOMODE,
        compute: {
            nodePools: ['system', 'general-purpose'],
   },
});
```

### Triển khai cụm EKS với nhóm nút được quản lý

<a href="https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html"><u>Nhóm Node Được Quản Lý của Amazon EKSS</u></a> mang đến trải nghiệm quản lý điện toán liền mạch cho các cụm Kubernetes của bạn. Khả năng mạnh mẽ này loại bỏ sự phức tạp trong vận hành bằng cách tự động hóa vòng đời đầu cuối của các phiên bản Amazon EC2, vốn là nền tảng
cho các ứng dụng được đóng gói trong container của bạn. Đằng sau hậu trường, nhóm node được quản lý của Amazon EKS sẽ sắp xếp những thay đổi này một cách thông minh, đảm bảo ứng dụng của bạn không bị gián đoạn nhờ khả năng thoát node mượt mà. Dịch vụ này tự động tận dụng các AMI mới nhất được tối ưu hóa cho Amazon EKS, mang đến một nền tảng an toàn và tối ưu cho khối lượng công việc của bạn.

Bằng cách thiết lập `defaultCapacityType` thành `NODEGROUP`, khách hàng có thể tận dụng phương pháp quản lý nhóm nút được quản lý truyền thống:

```typescript
Typescript
    import * as eksv2 from '@aws-cdk/aws-eks-v2-alpha';

    // Creating an EKS Cluster with Managed Node Groups and default instance types
    const eksCluster = new eksv2.Cluster(this, 'EksCluster', {
        version: eksv2.KubernetesVersion.V1_32,
        defaultCapacityType: eksv2.DefaultCapacityType.NODEGROUP,
});
```

Theo mặc định, khi sử dụng `DefaultCapacityType.NODEGROUP`, thư viện này sẽ phân bổ một nhóm nút được quản lý với hai `m5.large`phiên bản.
Sau khi triển khai mã trên, bạn có thể kiểm tra Bảng điều khiển EKS để xem cụm EKS đã được triển khai chưa như trong hình 3:

<p align="center">
  <img width="1641" height="722" alt="Image" src="https://github.com/user-attachments/assets/79cd1287-9a3a-4ac5-bb95-fd69950311c1" />
</p>
<p align="center"><em>Hình 3 – Cụm EKS được triển khai với các nhóm nút được quản lý</em></p>

Bạn cũng có thể kiểm tra tab Compute và xem Cấu hình nhóm nút được quản lý như trong hình 4:

<p align="center">
  <img width="1625" height="176" alt="Image" src="https://github.com/user-attachments/assets/7c069211-bec7-45c3-aef1-79bb772e2c22" />
</p>
<p align="center"><em>Hình 4 – Cấu hình mặc định của nhóm nút được quản lý của cụm EKS</em></p>

Nếu bạn muốn kiểm soát các loại phiên bản của Nhóm nút được quản lý, bạn có thể chỉ định loại EC2 mặc định làm thuộc tính của cấu trúc:
```typescript
Typescript
    import * as eksv2 from '@aws-cdk/aws-eks-v2-alpha';
    import * as ec2 from 'aws-cdk-lib/aws-ec2'

    // Creating an EKS Cluster with Managed Node Groups and specific instance types
    const eksCluster = new eksv2.Cluster(this, 'EksCluster', {
        version: eksv2.KubernetesVersion.V1_32,
        defaultCapacityType: eksv2.DefaultCapacityType.NODEGROUP,
        defaultCapacity: 5,
        defaultCapacityInstance: ec2.InstanceType.of(ec2.InstanceClass.M5, ec2.InstanceSize.SMALL),
});
```

Bạn cũng có thể chỉ định các tùy chỉnh bổ sung sau khi khai báo cụm EKS, thông qua `addNodegroupCapacity`phương thức:

```typescript
Typescript
    import * as eksv2 from '@aws-cdk/aws-eks-v2-alpha';
    import * as ec2 from 'aws-cdk-lib/aws-ec2'

    // Creating an EKS Cluster with Managed Node Groups and specific instance types
    const eksCluster = new eksv2.Cluster(this, 'EksCluster', {
        version: eksv2.KubernetesVersion.V1_32,
        defaultCapacityType: eksv2.DefaultCapacityType.NODEGROUP,
        defaultCapacity: 0,
});

    eksCluster.addNodegroupCapacity('custom-node-group', {
        instanceTypes: [new ec2.InstanceType('m5.large')],
        minSize: 4,
        diskSize: 100,
});
```

### Quản lý Quyền thông qua Mục Truy cập

Cấu trúc mới `aws-eks-v2`chuyển đổi từ xác thực dựa trên ConfigMap trước đây (đã bị loại bỏ trong EKS) sang <a href="https://aws.amazon.com/vi/blogs/devops/announcing-the-new-aws-cdk-eks-v2-l2-constructs/"><u>chế độ Xác thực Mục Truy cập</u></a>
. Thay đổi này giới thiệu Access Entry làm phương pháp chuẩn hóa để quản lý quyền cụm, cung cấp một phương pháp hợp lý và an toàn hơn để cấp quyền truy cập cụm cho người dùng và vai trò IAM.

Bạn có thể xác định Chính sách truy cập thông qua `AccessPolicy`cấu trúc và bạn có thể điều chỉnh phạm vi của Chính sách truy cập cho toàn bộ cụm EKS hoặc cho các Không gian tên EKS cụ thể:

```typescript
Typescript
    import * as eksv2 from '@aws-cdk/aws-eks-v2-alpha';

    // AmazonEKSClusterAdminPolicy with `cluster` scope
    eks.AccessPolicy.fromAccessPolicyName('AmazonEKSClusterAdminPolicy', {
      accessScopeType: eks.AccessScopeType.CLUSTER,
    });

    // AmazonEKSAdminPolicy with `namespace` scope
        eks.AccessPolicy.fromAccessPolicyName('AmazonEKSAdminPolicy', {
        accessScopeType: eks.AccessScopeType.NAMESPACE,
        namespaces: ['foo', 'bar'] 
    });
```

Sau đó, bạn có thể cấp quyền truy cập cho các Vai trò IAM cụ thể bằng grantAccessphương pháp:

```typescript
Typescript
    import * as iam from 'aws-cdk-lib/aws-iam'

    // Defining a IAM Role
    const clusterAdminRole = new iam.Role(this, 'ClusterAdminRole', {
    assumedBy: new iam.ArnPrincipal('arn_for_trusted_principal'),
    });

    // Creating an EKS Cluster with AutoMode
    const eksCluster = new eksv2.Cluster(this, 'EksCluster', {
    version: eksv2.KubernetesVersion.V1_32,
    defaultCapacityType: eksv2.DefaultCapacityType.AUTOMODE,
    });

    // Cluster Admin role for this cluster
    eksCluster.grantAccess('clusterAdminAccess', clusterAdminRole.roleArn, [
        eks.AccessPolicy.fromAccessPolicyName('AmazonEKSClusterAdminPolicy', {
            accessScopeType: eks.AccessScopeType.CLUSTER,
        }),
    ]);
```
Khi Principal đảm nhiệm vai trò `ClusterAdminRole`, nó sẽ nhận được quyền truy cập liền mạch vào cụm EKS thông qua một chuỗi cấp phép được sắp xếp cẩn thận. Quyền truy cập này được quản lý bởi `AmazonEKSClusterAdminPolicy`, được tự động đính kèm vào Chính sách Truy cập được liên kết với Vai trò IAM.

## Phần kết luận

<a href="https://docs.aws.amazon.com/cdk/api/v2/docs/aws-eks-v2-alpha-readme.html"><u>Trong bài viết này, chúng tôi đã giới thiệu cấu trúc AWS CDK L2</u></a> `(aws-eks-v2)` mới <a href="https://aws.amazon.com/vi/blogs/devops/announcing-the-new-aws-cdk-eks-v2-l2-constructs/"><u>dành cho Amazon EKS</u></a>, minh họa cách nó đơn giản hóa việc triển khai cụm đồng thời mang lại tính linh hoạt và hiệu quả vận hành vượt trội. Thông qua các ví dụ thực tế, chúng tôi đã trình bày cách khách hàng có thể tận dụng các mặc định thông minh và tùy chọn tùy chỉnh của cấu trúc này để xây dựng môi trường Kubernetes sẵn sàng cho môi trường sản xuất trên AWS.

Cấu trúc L2 mới dành cho Amazon EKS mang lại những cải tiến đáng kể giúp khách hàng đẩy nhanh hành trình áp dụng container:

- **Hiệu suất được nâng cao** : Loại bỏ sự phụ thuộc vào Tài nguyên tùy chỉnh và các chức năng <a href="https://docs.aws.amazon.com/lambda/latest/dg/welcome.html"><u>AWS Lambda bằng cách sử dụng tài nguyên AWS CloudFormation</u></a> gốc , mang lại khả năng triển khai nhanh hơn và đáng tin cậy hơn.

- **Xác thực hiện đại** : Triển khai xác thực dựa trên mục nhập truy cập, thay thế phương pháp ConfigMap đã lỗi thời bằng giải pháp an toàn và có thể lập trình được hơn.

- **Khả năng mở rộng được cải thiện** : Loại bỏ giới hạn cụm đơn trên mỗi ngăn xếp và loại bỏ các ngăn xếp lồng nhau, cho phép các mẫu kiến trúc linh hoạt hơn.

- **Tạo tài nguyên được tối ưu hóa** : Biến trình xử lý Lambda kubectl thành tùy chọn, giúp khách hàng kiểm soát chặt chẽ các thành phần cơ sở hạ tầng của mình.

- **Hoạt động hợp lý** : Cung cấp khả năng quản lý nhóm nút tự động với các mặc định thông minh trong khi vẫn duy trì toàn quyền kiểm soát của khách hàng khi cần.

Để bắt đầu với cấu trúc EKS L2 mới, hãy truy cập <a href="https://docs.aws.amazon.com/cdk/api/v2/docs/aws-eks-v2-alpha-readme.html"><u>tài liệu AWS CDK</u></a>. Nếu bạn có những tính năng cụ thể muốn bổ sung, chúng tôi khuyến khích bạn gửi yêu cầu tính năng trong <a href="https://github.com/aws/aws-cdk"><u>kho lưu trữ GitHub aws-cdk</u></a>. Phản hồi của bạn sẽ giúp chúng tôi tiếp tục đổi mới vì bạn.


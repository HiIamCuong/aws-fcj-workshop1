---
title : "Tải dữ liệu nguồn"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 2.1.2 </b> "
---

## Tạo tích hợp Zero-ETL
1. To create the zero-ETL integration
+ Truy cập **Amazon RDS** console
+ Chọn tính năng **Zero-ETL integrations** ở thanh bên trái
+ Chọn **Create zero-ETL integration**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/10.png)

2. Sau khi xong bước 1, Ở phần **Integration identifier**
+ Nhập vào tên Integration mà bạn muốn, for example `zero-etl-aurora-mysql-redshift`. 
+ Nhấn **Next**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/11.png)

3. Sau khi xong bước 2 
+ Nhấn Browse RDS databases
+ Chọn source aurora mysql cluster tên **zero-etl-lab-sourceamscluster-***.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/12.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/13.png)

- Vì các yêu cầu tiên quyết của nguồn dữ liệu chưa được đáp ứng, chúng ta sẽ nhận được một thông báo lỗi kèm theo một ô kiểm **Fix it for me (requires reboot)**. Bạn cũng sẽ thấy một tùy chọn để áp dụng **DATA FILTERING**. Thực hiện các hành động sau:

- Tích vào ô kiểm **Fix it for me (requires reboot)**.
- Tích vào ô kiểm **Customize data filtering options** dưới mục **Data filtering options**.
- Đối với loại bộ lọc **Include**, sử dụng `*.*` làm biểu thức bộ lọc. Bỏ qua nếu đã được điền sẵn.
- Nhấn vào nút **Add Filter** để thêm tiêu chí loại trừ như: **Exclude** các giá trị như `filter_missingpk.*`. Điều này sẽ ngăn không cho bất kỳ bảng nào trong schema/database có tên **filter_missingpk** được đồng bộ qua tích hợp zero-etl.
- Khi hoàn thành, nhấn **Next**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/14.png)

- **Fix it for me** bắt đầu hoạt động! Bạn sẽ thấy các thay đổi trên cluster nguồn sẽ được áp dụng. Nó sẽ tạo một *DbClusterParameterGroup* mới với tên **zero-etl-params-*** và tự động thiết lập 6 tham số với các giá trị yêu cầu. Điều này cũng sẽ yêu cầu một lần khởi động lại cluster của bạn, vì vậy bạn sẽ phải gõ **confirm** và nhấn **Reboot and continue**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/15.png)

+ Thanh trạng thái hiển thị quá trình thay đổi của cluster nguồn của bạn. Bạn có thể nhấn vào nút **View details** để xem các thay đổi.

+ Khi hoàn tất, thanh trạng thái sẽ chuyển sang màu xanh và nút **View details** sẽ hiển thị tất cả các thay đổi yêu cầu là *Complete*.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/18.png)

4. Sau bước 3 Select target 
+ Nhấn **Browse Redshift data warehouses**

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/19.png)

+ Chọn Redshift Serverless destination namespace **zero-etl-destination-***.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/20.png)

Vì các yêu cầu tiên quyết của đích đến chưa được thực hiện, chúng ta sẽ nhận được một thông báo lỗi kèm theo ô kiểm **Fix it for me** ở dưới.
+ Tích *Fix it for me*
+ Nhấn Next.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/21.png)

Bây giờ bạn sẽ thấy màn hình **Review changes**. Nó sẽ hiển thị cho bạn **Resource policy** và **thay đổi tham số phân biệt chữ hoa chữ thường**.

+ Bạn sẽ nhấn vào **Continue** và tiếp tục.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/22.png)

+ Thanh trạng thái sẽ hiển thị quá trình thay đổi của datawarehouse đích. Các thay đổi đích sẽ hoàn tất nhanh chóng và thanh trạng thái sẽ chuyển sang màu xanh. Bạn có thể nhấn vào nút **View details** để xem các thay đổi đã hoàn tất. Các thay đổi này diễn ra trong nền, không ngăn bạn tiếp tục thực hiện các bước tiếp theo.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/23.png)

5. Bước 4 Tùy chọn: Thêm tags và mã hóa
+ Không cần thực hiện hành động nào và nhấn **Next**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/24.png)

6. Bước 5 Xem lại và tạo, thực hiện các hành động sau:
+ Xem lại để đảm bảo tất cả các yêu cầu tiên quyết đã được xử lý bởi nút **Fix it for me**. Bạn có thể kiểm tra tiến trình của các thay đổi bằng cách nhấn vào nút **View Details** trong thanh tiến trình ở đầu bảng điều khiển. Bạn có thể phải đợi vài phút vì các thay đổi nguồn **cần khởi động lại** và sẽ mất vài phút.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/25.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/26.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/27.png)

Bạn có thể nhấn vào tích hợp để xem chi tiết và theo dõi tiến trình của nó. Quá trình này sẽ mất vài phút để thay đổi trạng thái từ **Creating** sang **Active**. Thời gian thay đổi phụ thuộc vào kích thước của dataset đã có sẵn trong nguồn.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/28.png)

Trong quá trình tạo tích hợp zero-ETL, trạng thái của Redshift serverless workgroup và namespace sẽ thay đổi từ **Available** sang **Modifying** rồi lại về **Available**. Đây là trạng thái bình thường và dự kiến.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/29.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/30.png)

{{% notice info %}}  
Quá trình tạo tích hợp zero-ETL sẽ mất khoảng **15-30 phút**. Hãy cùng thảo luận về tầm nhìn của zero-ETL và giải đáp bất kỳ câu hỏi nào cho đến nay. Bạn cũng có thể xem xét làm việc với các lab khác như **Streaming Ingestion** hoặc **Federated Query** trong lúc chờ đợi. **Lưu ý**: Bạn không thể tạo nhiều tích hợp zero-etl song song tới cùng một đích đến. Nếu một tích hợp đang trong trạng thái **Creating**, tích hợp tiếp theo phải đợi cho đến khi hoàn tất.
{{% /notice %}}

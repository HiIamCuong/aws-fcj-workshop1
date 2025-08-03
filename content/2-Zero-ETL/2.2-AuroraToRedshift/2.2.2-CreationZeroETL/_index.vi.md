---
title : "Khởi tạo Zero-ETL"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 2.2.2 </b> "
---

## Tạo Zero-ETL Integration
1. Để tạo Zero-ETL integration
+ Truy cập vào **Amazon RDS** console
+ Chọn **Zero-ETL integrations** trong bảng điều hướng
+ Nhấn vào **Create zero-ETL integration** như hình dưới.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/10.png)

2. Trong **Bước 1** Khởi động, đối với **Integration identifier**
+ Nhập một tên integration theo ý bạn, ví dụ `zero-etl-rds-mysql-redshift`. 
+ Nhấn **Next**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/63.png)

3. Trong **Bước 2** Chọn nguồn 
+ Nhấn vào **Browse RDS databases**
+ Chọn nguồn aurora mysql cluster có tên **zero-etl-lab-sourcerdsmscluster-**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/12.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/64.png)

Vì các yêu cầu trước cho nguồn chưa được đáp ứng, chúng ta sẽ nhận được thông báo lỗi cùng với một ô chọn **Fix it for me (requires reboot)**. Bạn cũng sẽ thấy một tùy chọn để áp dụng **DATA FILTERING**. Hãy thực hiện các bước sau:

+ Đánh dấu ô **Fix it for me (requires reboot)**.
+ Đánh dấu ô **Customize data filtering options** dưới **Data filtering options**.
+ Đối với loại bộ lọc **Include**, sử dụng `*.*` làm biểu thức lọc. Bỏ qua nếu đã được điền sẵn.
+ Nhấn vào nút **Add Filter** để thêm tiêu chí loại trừ như: **Exclude** các giá trị như `filter_missingpk.*`. Điều này sẽ ngăn các bảng trong lược đồ/cơ sở dữ liệu có tên **filter_missingpk** được truyền qua Zero-ETL integration.
+ Khi hoàn tất, nhấn **Next**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/14.png)

Fix it for me bắt đầu hoạt động! Bạn sẽ thấy các thay đổi trên cụm nguồn mà sẽ được áp dụng. Nó sẽ tạo một *ParameterGroup* mới với tên **zero-etl-params-*** và tự động cập nhật 2 tham số với các giá trị yêu cầu. Điều này cũng yêu cầu khởi động lại cụm của bạn, 
+ Vì vậy bạn cần nhấn **Reboot and continue**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/66.png)

+ Thanh trạng thái cho thấy cụm nguồn của bạn đang được chỉnh sửa. Bạn có thể nhấn vào nút **View details** để xem các thay đổi.

+ Khi hoàn tất, thanh trạng thái sẽ chuyển sang màu xanh và nút **View details** sẽ hiển thị tất cả các thay đổi yêu cầu dưới dạng *Complete*.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/67.png)

4. Trong **Bước 3** Chọn đích
+ Nhấn vào **Browse Redshift data warehouses**

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/19.png)

+ Chọn không gian tên Redshift Serverless đích **zero-etl-destination-***.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/20.png)

Vì các yêu cầu trước cho đích chưa được thực hiện, chúng ta sẽ nhận được thông báo lỗi cùng với một ô chọn **Fix it for me** bên dưới.
+ Đánh dấu ô *Fix it for me*
+ Nhấn Next.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/21.png)

Bây giờ bạn sẽ thấy màn hình **Review changes**. Màn hình này sẽ hiển thị **Resource policy** và **thay đổi tham số phân biệt chữ hoa chữ thường**. 

+ Bạn nhấn vào Continue và tiếp tục.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/69.png)

+ Thanh trạng thái sẽ cho thấy kho dữ liệu đích đang được chỉnh sửa. Các thay đổi đích sẽ hoàn tất nhanh chóng và thanh trạng thái chuyển sang màu xanh. Bạn có thể nhấn vào nút **View details** để xem các thay đổi đã hoàn tất. Các thay đổi sẽ diễn ra trong nền và không làm gián đoạn quá trình tiếp theo.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/23.png)

5. Trong bước tùy chọn **Bước 4 Thêm thẻ và mã hóa**
+ Không cần thực hiện hành động nào và nhấn **Next**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/24.png)

6. Trong **Bước 5 Xem lại và tạo**, thực hiện các bước sau
+ Xem lại để đảm bảo tất cả các yêu cầu đã được xử lý bằng nút **Fix it for me**. Bạn có thể kiểm tra tiến trình thay đổi bằng nút **View Details** trong thanh tiến trình ở trên cùng của console. Bạn có thể phải đợi một vài phút vì các thay đổi của nguồn **yêu cầu khởi động lại** mất vài phút.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/70.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/71.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/72.png)

Bạn có thể nhấn vào integration để xem chi tiết và theo dõi tiến trình của nó. Quá trình thay đổi trạng thái từ Creating sang Active mất vài phút. Thời gian thay đổi tùy thuộc vào kích thước của bộ dữ liệu đã có trong nguồn.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/74.png)

Trong quá trình tạo zero-ETL integration, trạng thái của Redshift serverless workgroup và không gian tên sẽ thay đổi từ Available sang Modifying rồi lại sang Available. Điều này là bình thường và mong đợi.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/29.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/30.png)

{{% notice info %}}  
Quá trình tạo zero-ETL integration sẽ mất khoảng **15-30 phút**. Hãy thảo luận về tầm nhìn của zero-ETL và những câu hỏi khác cho đến lúc này. Bạn cũng có thể xem xét làm các bài lab khác như **Streaming Ingestion** hoặc **Federated Query** trong thời gian này. **Lưu ý** bạn không thể tạo nhiều zero-etl integrations song song đến cùng một đích. Nếu một integration hiện tại đang **Creating**, integration tiếp theo sẽ phải đợi cho đến khi integration trước đó hoàn tất.
{{% /notice %}}

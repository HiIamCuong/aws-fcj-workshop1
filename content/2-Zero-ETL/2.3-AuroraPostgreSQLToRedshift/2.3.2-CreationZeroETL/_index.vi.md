---
title : "Khởi tạo Zero-ETL"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 2.3.2 </b> "
---

## Tạo Zero-ETL Integration

1. Để tạo Zero-ETL integration
+ Truy cập vào **Amazon RDS** console
+ Chọn **Zero-ETL integrations** trong bảng điều hướng
+ Nhấp vào **Create zero-ETL integration** như hình dưới đây.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/10.png)

2. Dưới **Bước 1** Bắt đầu, cho **Integration identifier**
+ Nhập tên cho integration, ví dụ `zero-etl-aurora-postgres-redshift`.
+ Nhấp vào **Next**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/117.png)

3. Dưới Bước 2 Chọn nguồn
+ Nhấp vào **Browse RDS databases**
+ Chọn cụm Aurora MySQL có tên **aurorapg-***.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/12.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/118.png)

Vì các yêu cầu trước khi kết nối chưa được đáp ứng, bạn sẽ nhận được thông báo lỗi kèm theo hộp kiểm **Fix it for me (requires reboot)**. Bạn cũng sẽ thấy tùy chọn để áp dụng **DATA FILTERING**. Thực hiện các bước sau:

+ Tích vào hộp kiểm **Fix it for me (requires reboot)**.
+ Tích vào hộp kiểm **Customize data filtering options** dưới **Data filtering options**.
+ Cho kiểu lọc **Include**, sử dụng `postgres.public.*` làm biểu thức lọc. Bỏ qua nếu đã được điền sẵn.
+ Nhấp vào nút **Add Filter** để thêm tiêu chí loại trừ như: **Exclude** các giá trị như `postgres.public.customer_info_missing_pk`. Điều này sẽ ngăn không cho bất kỳ bảng nào trong schema/database có tên **customer_info_missing_pk** được chuyển qua zero-etl integration.
+ Khi hoàn tất, nhấp vào **Next**.

{{% notice info %}}
**Data Filtering** sử dụng ký tự đại diện *** wildcard** có thể áp dụng cho các trường hợp **ALL or NOTHING** đối với **schemas** hoặc **tables**. Nếu bạn muốn so khớp chuỗi theo kiểu **starts with** hoặc **contains**, hãy sử dụng biểu thức chính quy trong các điều kiện **include** và **exclude** với cú pháp lọc [maxwell filter syntax](https://maxwells-daemon.io/filtering/). Đọc thêm trong tài liệu [data filtering for zero etl](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/zero-etl.filtering.html#zero-etl.filtering-format).
{{% /notice %}}

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/119.png)

Tùy chọn **Fix it for me** bắt đầu hoạt động! Bạn sẽ được trình bày với các thay đổi trên cụm nguồn sẽ được áp dụng. Nó sẽ tạo một *ParameterGroup* mới có tên **zero-etl-params-*** và tự động cập nhật 2 tham số với các giá trị cần thiết. Điều này cũng yêu cầu khởi động lại cụm của bạn.
+ Vì vậy, bạn cần nhập **confirm** trong hộp và nhấp vào **Reboot and continue**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/120.png)

+ Thanh trạng thái sẽ hiển thị cụm nguồn của bạn đang được thay đổi. Bạn có thể nhấp vào nút **View details** để xem các thay đổi.

+ Khi hoàn tất, thanh trạng thái sẽ chuyển sang màu xanh và nút **View details** sẽ hiển thị tất cả các thay đổi đã hoàn tất với trạng thái *Complete*.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/121.png)

4. Dưới Bước 3 Chọn mục tiêu
+ Nhấp vào **Browse Redshift data warehouses**

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/19.png)

+ Chọn không gian tên Redshift Serverless đích **zero-etl-destination-***.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/20.png)

Vì các yêu cầu trước khi kết nối cho mục tiêu chưa được thực hiện, bạn sẽ nhận được thông báo lỗi cùng với hộp kiểm **Fix it for me** bên dưới.
+ Tích vào **Fix it for me**
+ Nhấp vào **Next**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/122.png)

Bây giờ bạn sẽ được trình bày màn hình **Review changes**. Màn hình này sẽ hiển thị **Resource policy** và các thay đổi tham số phân biệt chữ hoa chữ thường.

+ Bạn sẽ nhấp vào **Continue** và tiếp tục.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/123.png)

+ Thanh trạng thái sẽ hiển thị kho dữ liệu đích của bạn đang được thay đổi. Các thay đổi đối với mục tiêu sẽ hoàn tất nhanh chóng và thanh trạng thái sẽ chuyển sang màu xanh. Bạn có thể nhấp vào nút **View details** để xem các thay đổi đã hoàn tất. Các thay đổi xảy ra trong nền và không ngăn cản bạn tiếp tục.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/124.png)

5. Dưới bước tùy chọn 4 Thêm thẻ và mã hóa
+ Không thực hiện hành động nào và nhấp vào **Next**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/24.png)

6. Dưới Bước 5 Xem lại và tạo, thực hiện các bước sau
+ Xem lại để đảm bảo tất cả các yêu cầu trước khi kết nối đã được xử lý bởi nút **Fix it for me**. Bạn có thể kiểm tra tiến độ của các thay đổi bằng cách sử dụng nút **View Details** trong thanh tiến độ ở trên cùng của bảng điều khiển. Bạn có thể phải đợi vài phút vì các thay đổi đối với nguồn **cần khởi động lại** và việc này sẽ mất vài phút.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/125.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/126.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/127.png)

Bạn có thể nhấp vào integration để xem chi tiết và theo dõi tiến độ của nó. Quá trình thay đổi trạng thái từ **Creating** sang **Active** mất vài phút. Thời gian này có thể thay đổi tùy thuộc vào kích thước của bộ dữ liệu đã có sẵn trong nguồn.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/128.png)

Trong quá trình tạo Zero-ETL integration, trạng thái của nhóm và không gian tên Redshift Serverless sẽ thay đổi từ Available sang Modifying rồi lại sang Available. Điều này là bình thường và đã được mong đợi.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/29.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/30.png)

{{% notice info %}}  
Quá trình tạo Zero-ETL integration sẽ mất khoảng **15-30 phút**. Hãy cùng thảo luận về tầm nhìn Zero-ETL và bất kỳ câu hỏi nào cho đến nay. Bạn cũng có thể xem xét làm việc trên các bài tập khác như **Streaming Ingestion** hoặc **Federated Query** trong thời gian chờ đợi. **Lưu ý** bạn không thể tạo nhiều Zero-ETL integrations đồng thời cho cùng một đích. Nếu một integration đang ở trạng thái **Creating**, integration tiếp theo phải chờ cho đến khi nó hoàn tất.
{{% /notice %}}

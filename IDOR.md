# Insecure direct object references (IDOR) - Tham chiếu đối tượng trực tiếp không an toàn

## 1. Tham chiếu đối tượng trực tiếp không an toàn (IDOR) là gì?

Insecure direct object references (IDOR) là một loại lỗ hổng kiểm soát truy cập, xuất hiện khi một ứng dụng sử dụng dữ liệu đầu vào của user để truy cập trực tiếp đến đối tượng. Thuật ngữ IDOR dần được nhiều người biết đến khi nó lần đầu xuất hiện trong Top 10 OWASP 2007. Tuy nhiên đây chỉ là một trong nhiều ví dụ về lỗi triển khai kiểm soát truy cập mà có thể dẫn đến việc bị lợi dụng. Các lỗ hổng IDOR thường liên quan đến việc leo thang đặc quyền theo chiều ngang, với một vài biến thể phát sinh leo thang đặc quyền theo chiều dọc.

Ví dụ về IDOR

Có nhiều ví dụ về lỗ hổng kiểm soát truy cập mà trong đó một giá trị tham số của người dùng sẽ được sử dụng để truy cập trực tiếp vào tài nguyên hoặc hàm.

### 1.1. Lỗ hổng IDOR với tham chiếu trực tiếp đến các đối tượng trong cơ sở dữ liệu

Lấy ví dụ về một trang web sử dụng URL sau để truy cập vào trang cá nhân của khách hàng, từ back-end của cơ sở dữ liệu:
```
https://insecure-website.com/customer_account?customer_number=132355
```
ID của khách hafg được sử dụng trực tiếp làm mã truy vấn để lấy thông tin từ cở sở dữ liệu. Nếu không có một quá trình nào khác can thiệp, Hacker chỉ cần thay đổi giá trị customer\_number là sẽ có thể xem được thông tin của các khách hàng khác. Đây là một ví dụ về lỗ hổng IDOR dẫn đến leo thang đặc quyền theo chiều ngang. 

Kẻ tấn công có thể thực hiện leo thang đặc quyền theo chiều ngang và chiều dọc bằng cách đổi user thành user với đặc quyền khi lợi dụng lỗ hổng. Ngoài ra tùy trường hợp mà các lỗ hổng như lộ mật khẩu và thay đổi tham số cũng có thể được sử dụng khi kẻ tấn công đã truy cập được vào trang tài khoản người dùng.

### 1.2. Lỗ hổng IDOR khi tham chiếu trực tiếp tới tệp tin

Một hình thức dễ lợi dụng khác là khi các tài nguyên quan trọng được lưu trong các tệp tin tĩnh trên server. Ví dụ một website lưu các đoạn tin nhắn vào ổ đĩa với tên được đánh số thứ tự tăng dần, và user có thể truy cập vào chúng bằng URL như sau:
```
https://insecure-website.com/static/12144.txt
```
Trong tình huống này, người tấn công chỉ cần thay đổi tên file là đã có thể truy cập vào đoạn chat tạo bởi một user khác, và có nguy cơ biết được danh tính và nhiều dữ liệu quan trọng khác.


Để kiểm tra lỗ hổng này, người kiểm thử cần tìm mọi tác vụ trên ứng dụng mà tham số người dùng nhập vào được sử dụng để tham chiếu trực tiếp tới đối tượng. Ví dụ, các địa điểm mà tham số người dùng nhập vào được dùng để truy cập vào cơ sở dữ liệu, một tập tin, một trang ứng dụng,… Tiếp theo, người kiểm thử sẽ thực hiện thay đổi giá trị của tham số được dùng để tham chiếu đối tượng và đánh giá mức độ khả thi của việc đánh cắp dữ liệu từ người dùng khác hoặc bypass thẩm quyền.

Cách tốt nhất để kiểm thử tham chiếu đối tượng trực tiếp là với ít nhất 2 (hoặc nhiều hơn) người dùng để thử nghiệm với tài nguyên riêng mỗi người dùng được sở hữu. Ví dụ với 2 người dùng được truy cập vào các đối tượng khác nhau (thông tin mua bán, tin nhắn riêng tư,…) và sở hữu đặc quyền khác nhau (ví dụ như admin  - Quản trị viên) để kiểm tra có tồn tại việc tham chiếu trực tiếp tới ứng dụng, tác vụ, đối tượng hay không. Việc có nhiều user để thử nghiệm sẽ giúp rút ngắn quá trình kiểm thử thay vì phải tự ngồi đoán mò tên hoặc id của các đối tượng.

## 2. Một vài tình huống phổ biến liên quan tới lỗ hổng này và cách để kiểm thử chúng.

### 2.1. Giá trị của tham số được sử dụng trực tiếp để lấy thông tin từ Database
```
http://foo.bar/somepage?invoice=12345
```
giá trị của tham số invoice được dùng làm con trỏ trong một table chứa nhiều hóa đơn trong cơ sở dữ liệu. Hệ thống sẽ lấy giá trị này, đối chiếu với cơ sở dữ liệu và trả về thông tin cho người dùng.

Trong trường hợp này, thay đổi giá trị của tham số có thể sẽ giúp ta truy cập được bất kỳ đối tượng invoice nào trong cùng table. Để kiểm tra hiệu quả thì người kiểm thử nên có thông tin về invoice của nhiều user khác nhau, cần xác định rằng họ không nên có quyền xem invoice của người khác. Khi đó việc khai thác lỗ hổng sẽ được xếp vào truy cập tài nguyên một cách trái phép

### 2.2. Giá trị của tham số được sử dụng trực tiếp để thực hiện một hoạt động trong hệ thống
```
http://foo.bar/changepassword?user=someuser
```
Trong trường hợp nyaf, giá trị của tham số user được dùng để báo cho ứng dụng biết nó cần thực hiện việc đổi password cho user nào. 

Lúc này người kiểm thử có thể thực hiện đổi giá trị bằng tên các user khác nhau để xem tính khả thi khi đổi user của người khác ngoài quyền hạn của user hiện tại

### 2.3. Giá trị của tham số được sử dụng trực tiếp để truy cập vào tài nguyên của hệ thống tập tin
```
http://foo.bar/showImage?img=img00011
```
trong trường hợp này, giá trị của tham số file được dùng để cho ứng dụng biết người dùng đang cần lấy file nào. Thay đổi tên file có thể giúp kẻ tấn công lấy được những tệp tin khác thuộc sở hữu của các user khác.

Lỗ hổng này thường được khai thác cùng với lỗ hổng directory/path traversal

### 2.4. Giá trị của tham số được sử dụng trực tiếp để truy cập vào chức năng của ứng dụng
```
http://foo.bar/accessPage?menuitem=12
```
trong trường hợp này, giá trị của tham số menuitem được dùng để khai báo cho ứng dụng item nào trong menu mà người dùng đang muốn truy cập. Ví dụ về mặt pháp lý, 1 user chỉ có quyền dùng item 1, 2 và 3. Vậy khi thay đổi tham số này, có khả năng họ sẽ vi phạm chính sách và truy cập những thứ ngoài thẩm quyền họ có. 

Các ví dụ bên trên đều được lấy trường hợp request chỉ có 1 tham số. Nhưng cũng có trường hợp mà nhiều tham số được sử dụng, lúc này người kiểm thử cần linh hoạt và tùy biến

## 3. Tìm IDOR ở những nơi không ngờ tới
### 3.1. Kiểm tra ID đã mã hóa hoặc bị băm
Khi gặp một ID được mã hóa, có thể kiểm tra khả năng giải mã ID đã mã hóa bằng các phương pháp mã hóa thông thường. 

Nếu ứng dụng sử dụng một ID băm / ngẫu nhiên, có thể thử đoán cấu trúc ID không. Đôi khi các ứng dụng sử dụng các thuật toán xáo trộn ID là không đủ, vì vậy các ID có thể được dự đoán sau khi phân tích cẩn thận. Trong trường hợp này, hãy tạo thêm vài tài khoản để phân tích cách tạo ra các ID này. Từ đó dẫn đến khả năng tìm được cấu trúc của ID để lợi dụng.

Ngoài ra, các ID ngẫu nhiên hoặc đã băm có thể được tìm thấy thông qua các API  endpoint khác, trên các trang công khai khác trong ứng dụng (trang hồ sơ của người dùng khác, vv), hoặc trong URL thông qua trình tham chiếu.

Ví dụ một trường hợp kiểm thử như sau, một API endpoint cho phép người dùng lấy chi tiết tin nhắn trực tiếp thông qua một ID đã băm. Request có dạng:
```http
GET /api\_v1/messages?conversation_id=SOME_RANDOM_ID
```
Conversation\_id là một đoạn mã ngẫu nhiên dài, chứa cả chữ và số. Nhưng sau đó người kiểm thử phát hiện ra rằng có thể tìm được danh sách mọi tin nhắn của người dùng với ID của họ!
```http
GET /api\_v1/messages?user_id=ANOTHER_USERS_ID
```
Request này trả về 1 danh sách *conversation\_ids* mà người dùng có. Trong đó *user\_id*  của người dùng lại được công khai ở trang profile của người dùng tương ứng. Tức là chúng ta có thể đọc tin nhắn của bất kỳ người dùng nào sau khi có được user\_id của họ lấy từ trang profile công khai, rồi dùng user\_id đó để lấy danh sách conversation\_ids của họ, Cuối cùng yêu cầu xem tin nhắn thông qua API endpoint /api\_v1/messages!
### 3.2. Nếu không đoán được thì tự chế ra xem
Nếu ID dùng để tham chiếu đối  tượng quá phức tạp, hãy thử phân tích và lợi dụng các hàm đã tạo ra ID hoặc link ID đến đối tượng
### 3.3. Nhét ID vào mồm bắt phải nhận
Nếu request của ứng dụng không dùng ID nào cả, thì ta có thể thử tự thêm vào. Thử với các từ khó phổ biến như *id, user\_id, message\_id* hoặc các tham số tham chiếu đối tượng khác và quan sát xem chương trình có thay đổi gì hay không.

Ví dụ, nếu request này hiện tất cả tin nhắn của người dùng hiện tại:
```http
GET /api_v1/messages
```
Vậy biết đâu khi thêm user_id như dưới đây thì sẽ hiện tin nhắn của người dùng khác?
```http
GET /api_v1/messages?user_id=ANOTHER_USERS_ID
```
### 3.4. HPP (HTTP parameter pollution)
HPP vulnerabilities (gán nhiều giá trị cho cùng một tham số) cũng có thể dẫn tới IDOR. Nhiều ứng dụng sẽ không lường trước tình huống người dùng nhập nhiều giá trị cho cùng một tham số, do đó ta sẽ có khả năng bypass chương trình.

Tuy nhiên trường hợp này khá là hiếm khi xảy ra, còn trên lý thuyết thì sẽ trông như thế này. Nếu như request này fail:
```http
GET /api_v1/messages?user_id=ANOTHER_USERS_ID
```
Thì ta thử:
```http
GET /api_v1/messages?user_id=YOUR_USER_ID&user_id=ANOTHER_USERS_ID
```
Hoặc là:
```htt
GET /api_v1/messages?user_id=ANOTHER_USERS_ID&user_id=YOUR_USER_ID
```
Hoặc thay tham số thành dạng list:
```http
GET /api_v1/messages?user_ids[]=YOUR_USER_ID&user_ids[]=ANOTHER_USERS_ID
```
### 3.5. Blind IDORs
Đôi khi, các endpoint tương tác với IDOR không phản hồi trực tiếp với thông tin bị rò rỉ. Thay vào đó, chúng có thể khiến ứng dụng rò rỉ thông tin ở nơi khác: trong file xuất ra, email và thậm chí có thể là cảnh báo bằng văn bản.
### 3.6. Thay đổi request method
Nếu một request method không có tác dụng, có thể thay đổi các method khác như `GET`, `POST`, `PUT`, `DELETE`, `PATCH`...

Một mánh phổ biến là thay thế `POST` bằng `PUT`,… 
### 3.7. Thay đổi định dạng file
Đôi khi việc thay đổi định dạng file lấy từ server có thể dẫn đến việc server sẽ xử lý quyền của yêu cầu đó theo cách khác có thể sẽ khai thác được. Ví dụ thử thêm đuôi .json vào cuối request URL và xem response trả về.
## 4. Cách gia tăng ảnh hưởng của lỗ hổng IDOR
### Critical IDORs first
Thử tìm lỗ hổng IDOR trong các function quan trọng trước, ví dụ như IDOR dựa trên việc đọc/ghi đều có thể gây ra hậu quả nghiêm trọng.

Với lỗ hổng IDOR thay đổi trạng thái (ghi), reset password, thay đổi password hoặc recovery tài khoản thường có tác động lớn nhất tới hệ thống.

Với lỗ hổng IDOR bất thay đổi trạng thái (đọc), hãy thử tìm các chức năng liên quan đến thông tin nhạy cảm của ứng dụng. Ví dụ như các chức năng làm việc với tin nhắn, thông tin cá nhân, nội dung riêng tư. Sau đó tìm và khai thác lỗ hổng IDOR của chúng.

## 5. Hậu quả

Truy cập thông tin trái phép

Thay đổi hoặc xóa bỏ dữ liệu ngoài quyền hạn

Thực hiện các tác vụ khác ngoài quyền hạn

## 6. Cách phòng chống

Cấu hình các chính sách kiểm soát truy cập để người dùng không thể thực hiện các hành động ngoài quyền hạn

Sử dụng các hàm băm và giá trị băm thay vì giá trị thông thường của tham số

![](Aspose.Words.435cf7aa-b9e0-421c-9564-ae2d1230c3cd.001.png)


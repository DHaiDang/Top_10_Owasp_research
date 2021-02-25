# 🧐 SQL Injection<br>


## 🧐 SQLI <a name = "about"></a>

Là kỹ thuật lợi dụng những câu truy vấn của ứng dụng, chèn vào những câu sql để có thể
làm thay đổi cấu trúc truy vấn ban đầu, có thể thay đổi dữ liệu cơ sở dữ liệu, dẫn đến việc mình có thể như qủản trị website đó.


## 🏁 Các loại sqli và các nhận biết <a name = "getting_started"></a>

<br>

### 🧐 InBand SQLI : Attacker có thể tấn công và lấy đưọc dữ liệu ở trang web đó. 2 kiểu tấn
công kiểu in band thông thuờng là :
- Error based sqli : Khi mà nhập những đoạn hay kí tự mà nhận đưọc lỗi mà database
server ném ra message thì có thể dựa vào chỗ đó để chèn những câu sql vào tấn công.
- Union based sqli : Dùng những câu query UNION Select để có thể đưa ra data của
server và nó trả về như là 1 response của http.
<br>

### 🧐 Blind sqli : Tốn nhiều thời gian để khai thác hơn. là cách mà attacker sẽ thay đổi cấu trúc của database rồi đánh giá dựa vào response của database server để tìm cách tấn công
- Boolean based sqli : là kĩ thụât tấn công dựa trên phản hồi true hoặc false mà sẽ làm
cho http response thay đổi hoặc không thay đổi. Dựa vào đó mà attacker có thể payload
đoán xem là khi nào mình tấn công đúng thì sẽ thay đổi response chả hạn. :v
- Time based sqli : Cũng na ná cách boolean based, nhưng mà điều kiện để biết là
đúng sai tuỳ thuộc vào thời gian response của server, ví dụ dùng sleep cho những truờng
hợp tấn công đúng thì time response sẽ đợi lâu hơn 1 chút và attacker có thể dựa vào đó
mà biết là mình đang làm đúng hay làm sai.
### 🧐 Out-of- band Sqli :Kĩ thuật này ít dùng, thuờng khi mà attacker không dùng đưọc các cách mà kết quả trả về ngay trên trang đó thì họ.Kĩ thuật này dựa vào database server tạo DNS, http request cung cấp data cho attacker. Ví dụ Mysql có lệnh xp_dirtree có thể tạo dns request đến server của attacker.

<br>

## 🔧 Các cách lập trình an toàn để không bị sqli (php):
### - Ép kiểu dữ liệu thành Int :
- Ví dụ với url như thế này :
```
    domain.com/index.php?id=12
```
ta có thể ép kiểu thế này
```
$id = isset($_GET['id'])?(string)(int)$_GET['id']:false;
```
thì những kí tự sẽ bị loại ra.
Hoặc dùng hàm replace để thay những kí tự bằng số :
```
$id = str_replace('/[^0-9]/','',$id);
```
### - Sử dụng hàm mysql_real_escape_string để chuyển thành chuỗi query an toàn hơn sau đó kết hợp với hàm sprintf :
```
$sql = "SELECT * FROM member WHERE username = '%s' AND password ='%s'";
sprintf($sql,mysql_real_escape_string("donghaidang"),mysql_real_escape_string("pass
word"));
```
### - Không hiển thị lỗi, message (throw) : attacker có thể dựa vào lỗi bắn ra mà tấn công,nên có thể ẩn những đoạn throw lỗi đi

### - Phân quyền cẩn thận :
```
domain.com/index.php?type=vga'union select 1,2,load_file('/.passwd'),4;-- -
```
nếu để user truy vấn là root thi có thể query thẳng vào trong các file quan trọng, nên tạo user hạn chês quyền rồi truy vấn sau.

### - Tham số hoá (Nodejs):
```
'select * from users where email = ?' ,[email]
```
Đoạn này tuy truyền email vào nhưng [email] đã thành darta chứ không dùng cách nối chuỗi thông thường nên sẽ không xảy ra lỗi. <br>
Còn như thế này, attacker có thể thêm câu query vào và nó đưọc nối trực tiếp với câu
query của mình nên sẽ bị tấn công ngay.
```
'select * from users where email ="' + email + '";'
```
### - Web application firewall : là 1 tool hữu dụng để lọc sqli. Model mới nhất có chức năng lọc và chặn request đáng nghi

## 🏁 Các Kĩ thuật tấn công nâng cao (hơn vài cái ví dụ tý thui) <a name = "getting_started"></a>
- ### Bypass filter trong trường hợp ngta replace dấu cách
```
    $id = str_replace('','',$_GET('id'));
    $sql= "select * from products where id ='".$id."'";
    ...
```
Trong trường hợp này thì, thay dấu + bằng "\t" mặc dù thay đưọc dấy space thì đoạn \t vẫn khônh bị thay đổi,
đoạn code sẽ trở thành :
```
id'    union    select ...
```
vì là 1tab = 4 space nhưng mà câu query không quan trọng đến khoảng cách nên là câu query vẫn đúng.

- ### Optimize attack sqli boolean based : tối ưu hóa tấn công blind boolean based
    Trong khi payload data,
nhận về response và đoán là nó là kí tự nào, thì ta có thể lowercase tất cả rồi
chuyển thành mã ascii để compare,
từ đó các kí tự có thể rút xuống thành 26 chữ thôi (vì từ a-z) . khi đã có thứ tự
thành 26 số từ 97-122
thì ta dùng binary tree để tìm ra kí tự đúng.
ex : query : 
```
ascii(lower(password(name, 1, 1)))
```
Sau đó trong source code python ta sẽ implement binary search để tìm phần tử
theo dãy số.
query :
```
(
SELECT TOP 1 ascii(lower(substring(password, 1, 1))) FROM users
    WHERE id=(
    SELECT TOP 1 id FROM (
    SELECT TOP 1 id FROM users ORDER BY id)
    AS student ORDER BY id DESC)
) > 109
```
Tìm kiếm nhị phân sẽ chạy từ giữa là kí tự 'm' : 109 , như vậy sẽ có độ phức tạp
là 2logn = 2log26 ~ 5 lần chạy cho 1 kí tự.
- ### Tấn công sql nâng cao từ SQli đọc, ghi file (tự search nhá :v nhiều vãi :v )
<br>

## ⛏️ Luyện tập các lab để nâng cao trình độ <a name = "built_using"></a>
- [PortSwigger](https://portswigger.net/web-security/sql-injection)

<br>

## 🎉 Tips <a name = "acknowledgement"></a>

- Sử dụng tool sqlmap

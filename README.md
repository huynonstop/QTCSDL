# Xây dựng CSDL
 
## Tạo Database
 
```mssql
create database <name>
on
	primary
	(
    	name = "Primary_file",
        filename = "<path>.mdf",
        size = "MB",
        maxsize = "MB",
        filegrowth "MB/%"
    ), (
    	name = "secondary_file",
        filename = "<path>.ndf",
        size = "MB",
        maxsize = "MB",
        filegrowth "MB/%"
    )
    ,...
log on
	(
    	name = "log_file",
        filename = "<path>.ldf",
        size = "MB",
        maxsize = "MB",
        filegrowth "MB/%"
    )
```
 
## Sửa Database
 
### Xóa
 
```mssql
drop database <name>
```
 
### Sửa tên
 
```mssql
alter database <name>
modify name = <new name>
go
```
 
### Thêm file
 
```mssql
alter database <name>
add file (
	name = <name>,
    filename="path",
    size = "MB",
    maxsize = "MB"
)
```
 
### Sửa file
 
```mssql
alter database <name>
modify file (
	name = <name>,
    newname = <new name>,
    filaname = <path>, (for cutting)
    size = "MB"
)
```
 
### Xóa file
 
```mssql
alter database <name>
remove file <name>
```
 
## Detach database
 
```mssql
exec sp_detach_db <dbname>
```
 
## Attach database
 
```mssql
exec sp_attach_db 
<dbname>,
<file path>, <file path>
```
 
```mssql
create database <name>
on
	(filepath),
	(filepath)
for attach
```
 
## Kiểu dữ liệu tự tạo
 
```mssql
exec sp_addtype MSSV, varchar(7), "NOT NULL"
```
 
# Store procedure
 
## Biến
 
```mssql
declare @tenbien = <kieu dulieu>
set @tenbien = ...
```
 
## Switch
 
```mssql
case
when condition then dosomething
when condition then dosomething
else
```
 
## If
 
```mssql
if condition then
begin
	--do something
end
```
 
## Store procedure
 
```mssql
create procedure usp_name
@param type
as
	begin
 
	end
go
```
 
## Function
 
```mssql
create function uf_name (@param type, ...)
returns type
as
	begin
 
	end
go
```
 
```mssql
create function uf_name(@param type, ...)
return table
as
	begin
		return (select...)	
	end
go
```
 
## Cursor
 
```mssql
declare name cursor for (
	select...
)
open name
fetch next from cursor into @var, @var,...
while (@@fetchstatus = 0)
begin
	fetch next from cursor into @var, @var,...
end
close name
deallocate name
```
 
# Trigger
 
```mssql
create trigger on table_name
for insert, delete, update
as
	begin
 
	end
go
```
 
 
 
# Truy xuất bất đồng bộ
 
## 4 loại lỗi
 
1. Lost update: đọc cùng lúc, update sai
2. Dirty read: đọc dữ liệu bị sai do rollback
3. Unrepeatable read: có update nhưng không đọc lại
4. Phantom: có thêm, xóa, sửa trong khi xử lí tập data
 
## Giao tác
 
## Khóa
 
- Khóa đọc: S -> shared: nhiều đứa cùng đọc
- Khóa ghi: X -> X: 1 đứa được đọc
 
- Trước khi đọc: S or X
- Trước khi ghi: X
 
## Mức cô lập (isolation level)
 
1. Read uncommited:
2. Read commited: tránh dirty read (default)
3. Repeatable read: tránh dirty read, unrepeatable read
4. Serializable: tránh dirty read, unrepeatable read, phantom
 
## Cấp độ khóa
 
- NoLock
- ReadCommitted
- RepeatableRead
- Serializable
- Uplock
- Xlock
 
Example:
 
```mssql
create procedure usp_capNhatDiem
@mssv varchar(10),
@maMon varchar(10)
@diem varchar(10)
as
	begin
		begin tran
		set transaction isolation level serializable
		update KetQua
		set Diem = @diem
		where MSSV = @mssv and MaMon = @maMon
		if @@error
		begin
			rollback tran
			return
		end
		else
		begin
			commit tran
		end
	end
go
```
# Quản lý quyền người dùng
 
Login: (sysadmin/securityadmin)
 
## Tạo login 
 
exec sp_addlogin '@loginname', '@pass', '@defaultDB'
 
## Grant login cho người dùng windows
 
exec sp_grantlogin 'window_account'
exec sp_grantlogin 'Server01\user01'
 
## Đổi pass
 
exec sp_password '@oldpass','@newpass','@loginname'
 
## Hủy quyền login được grant
 
exec sp_revokelogin '@loginname'
 
## Xóa login
 
exec sp_droplogin '@loginname'
 
## Đổi db mặc định
 
exec sp_defaultdb '@loginname','@dbname'
 
User: (sysadmin,db_owner,db_accessadmin)
 
## Tạo user (cấp quyền truy cập cho 1 login vào database hiện hành)
 
Create user user_name 
For login login_name
 
Create user user_name 
From login login_name
With default schema schema_name
 
## Xóa user
 
exec sp_revokedbaccess '@username'
 
Role: (sysadmin, db_owner, db_securityadmin)
 
## Thêm role
 
exec sp_addrole '@rolename'[,'@owner']
exec sp_addrole 'Developer'
exec sp_addrole 'Developer','dbo'
 
## Xóa role
 
exec sp_droprole '@rolename'
 
## Thêm login vào role hệ thống
 
exec sp_addsrvrolemember '@loginname','@rolename'
 
## Thêm login vào role database
 
exec sp_addrolemember '@rolename','@username/@rolename'(@sercurity_acc)
 
## Server role
 
sysadmin : full quyền  (sa)
securityadmin : quản lý login (reset pass,grant,revoke,deny)
dbcreatetor: create,drop,alter,restore database
 
## Database role
 
db_owner : full quyền trong db (dbo)
db_accessadmin : add remove các truy cập
db_securityadmin : quản lí quyền và role trong db
db_datareader : đọc trên db
db_writter : ghi trên db
 
## Quyền thực hiên lệnh (create database, create procedure, ...)
 
### Cấp quyền
 
Grant ALL || quyền To @sercurity_acc
 
Grant create table, create procedure to dev01
 
### Từ chối (Thu hồi)
 
Deny ALL || quyền To @sercurity_acc
 
Deny create table, create procedure to dev01
 
### Lấy lại
 
Revoke ALL || quyền From  @sercurity_acc
 
Revoke create table from dev01
 
## Quyền thao tác trên đối tượng (đọc/ghi, thực hiện thủ tục, ...)
 
### Cấp quyền
 
GRANT ALL || quyền
ON table/view(column,...) || ON store_proc || ON function
TO @sercurity_acc
[WITH GRANT OPTION] (cho phép cấp quyền này cho các user/role khác)
[AS role] (cấp quyền này dưới tư cách là thành viên của role)
 
Grant select,update
On SinhVien(Hoten,DiaChi)
To Developer
 
Grant select,update
On SinhVien(Hoten,DiaChi)
To Developer
With grant option
 
### Từ chối (Thu hồi)
 
DENY ALL || quyền
ON table/view(column,...) || ON store_proc || ON function
TO @sercurity_acc
[CASCADE] (từ chối quyền này cho các user/role khác được cấp bởi @sercurity_acc này)
 
Deny select,update
On SinhVien(Hoten,DiaChi)
To Dev02
CASCADE
 
### Lấy lại
 
REVOKE ALL || quyền
ON table/view(column,...) || ON store_proc || ON function
FROM @sercurity_acc
[CASCADE]
[AS role]
 
Revoke select,update
On SinhVien(Hoten,DiaChi)
From Dev02
CASCADE
 
Revoke update
On SinhVien(Hoten,DiaChi)
From Developer

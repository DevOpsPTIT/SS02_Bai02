# Báo Cáo Thực Hành: Khởi Tạo User Thường & Thiết Lập Đặc Quyền Quản Trị (Sudoers Configuration)

- **Bài tập**: Bài 2 - Khởi tạo User thường và thiết lập đặc quyền quản trị (Session 02 / Linux Security & Privilege Management)
- **Đường dẫn nộp bài**: `SS02/Bai02/README.md`
- **Tài khoản khởi tạo**: `devops` (Cấp quyền `sudo`)

---

## 1. Bối cảnh & Mục tiêu

### Bối cảnh
Trong quản trị hệ thống Linux, việc thường xuyên đăng nhập và thao tác trực tiếp bằng tài khoản `root` tối cao tiềm ẩn rủi ro rất lớn (như lỡ tay xóa tệp tin hệ thống hoặc bị khai thác lỗ hổng bảo mật). Áp dụng **nguyên tắc đặc quyền tối thiểu (Principle of Least Privilege)**, quản trị viên cần tạo một tài khoản người dùng thường tên là `devops` cho các công việc hàng ngày và chỉ nâng quyền `sudo` khi cần thiết.

### Mục tiêu
- Tạo tài khoản người dùng thường `devops` trên máy chủ Ubuntu.
- Thêm người dùng `devops` vào nhóm quản trị `sudo`.
- Sao chép cấu hình SSH Public Key từ tài khoản `root` sang tài khoản `devops`.
- Phân quyền bảo mật chuẩn (`700` cho thư mục `.ssh` và `600` cho tệp `authorized_keys`).
- Thử nghiệm đăng nhập trực tiếp bằng SSH qua tài khoản `devops` và kiểm tra lệnh `sudo whoami`.

---

## 2. Môi trường triển khai

- **Hệ điều hành Server**: Ubuntu 22.04 LTS (DigitalOcean Droplet / VPS / WSL2)
- **Tài khoản thực hiện ban đầu**: `root`
- **Tài khoản khởi tạo mới**: `devops`
- **Phương thức đăng nhập**: SSH Key Ed25519

---

## 3. Các bước thực hiện chi tiết

### Bước 1: Tạo tài khoản người dùng `devops`

Đăng nhập vào máy chủ bằng quyền `root` và chạy lệnh tạo người dùng:

```bash
# Tạo user devops và tự động tạo thư mục home /home/devops
sudo adduser devops
```

*(Nhập mật khẩu an toàn cho user `devops` và điền các thông tin cá nhân tùy chọn).*

---

### Bước 2: Cấp đặc quyền quản trị `sudo` cho người dùng `devops`

Thêm `devops` vào nhóm hệ thống `sudo`:

```bash
# Thêm user devops vào nhóm sudo
sudo usermod -aG sudo devops

# Kiểm tra lại danh sách nhóm của user devops
groups devops
# Output mong đợi: devops : devops sudo
```

---

### Bước 3: Sao chép cấu hình SSH Key từ `root` sang `devops`

Sử dụng tiện ích `rsync` để sao chép thư mục `~/.ssh` từ `root` sang `/home/devops/` kèm thuộc tính phân quyền sở hữu:

```bash
# Sao chép toàn bộ cấu hình SSH Key và tự động đổi owner sang devops:devops
sudo rsync --archive --chown=devops:devops ~/.ssh /home/devops/
```

---

### Bước 4: Phân quyền bảo mật nghiêm ngặt cho thư mục SSH

OpenSSH yêu cầu các thư mục và tệp tin xác thực phải tuân thủ đúng phân quyền bảo mật (nếu sai phân quyền, SSH Daemon sẽ tự động từ chối kết nối):

```bash
# 1. Phân quyền 700 (rwx------) cho thư mục .ssh
sudo chmod 700 /home/devops/.ssh

# 2. Phân quyền 600 (rw-------) cho tệp authorized_keys
sudo chmod 600 /home/devops/.ssh/authorized_keys

# 3. Phân quyền sở hữu toàn bộ cho user devops
sudo chown -R devops:devops /home/devops/.ssh
```

---

## 4. Trích Xuất Log Terminal Chứng Minh (Verification Logs)

### 1. Thao tác khởi tạo user và phân quyền trên Server (Tài khoản `root`)

```console
root@ubuntu-devops-singapore:~# adduser devops
Adding user `devops' ...
Adding new group `devops' (1001) ...
Adding new user `devops' (1001) with group `devops' ...
Creating home directory `/home/devops' ...
Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for devops
Enter the new value, or press ENTER for the default
        Full Name []: DevOps Admin
        Room Number []: 
        Work Phone []: 
        Home Phone []: 
        Other []: 
Is the information correct? [Y/n] y

root@ubuntu-devops-singapore:~# usermod -aG sudo devops
root@ubuntu-devops-singapore:~# groups devops
devops : devops sudo

root@ubuntu-devops-singapore:~# rsync --archive --chown=devops:devops ~/.ssh /home/devops/
root@ubuntu-devops-singapore:~# chmod 700 /home/devops/.ssh
root@ubuntu-devops-singapore:~# chmod 600 /home/devops/.ssh/authorized_keys
root@ubuntu-devops-singapore:~# ls -la /home/devops/.ssh
total 12
drwx------ 2 devops devops 4096 Oct  4 20:18 .
drwxr-x--- 4 devops devops 4096 Oct  4 20:18 ..
-rw------- 1 devops devops  572 Oct  4 20:18 authorized_keys
```

---

### 2. Kiểm tra đăng nhập SSH từ Máy Cá Nhân bằng tài khoản `devops`

Mở Terminal trên máy cá nhân và thực hiện kết nối:

```console
user@local-pc:~$ ssh -i ~/.ssh/id_ed25519 devops@128.199.200.100
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-105-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: Sun Oct  4 20:18:10 2026 from 113.161.xx.xx
devops@ubuntu-devops-singapore:~$ whoami
devops
devops@ubuntu-devops-singapore:~$ pwd
/home/devops
```

---

### 3. Kiểm tra đặc quyền `sudo` (`sudo whoami`)

Chạy lệnh kiểm tra quyền quản trị:

```console
devops@ubuntu-devops-singapore:~$ sudo whoami
[sudo] password for devops: 
root
```

> **Đánh giá**: 
> 1. Đăng nhập thành công bằng SSH Key qua tài khoản người dùng thường `devops`.
> 2. Thực thi lệnh `sudo whoami` thành công và nhận về kết quả `root`.

---

## 5. Kết luận

- Đã khởi tạo thành công tài khoản làm việc an toàn `devops` với đặc quyền quản trị `sudo`.
- Phân quyền chính xác các tệp tin SSH Key (`700` cho `.ssh` và `600` cho `authorized_keys`), đáp ứng đầy đủ tiêu chuẩn an toàn bảo mật Linux.
- Hệ thống sẵn sàng thao tác quản trị mà không cần đăng nhập trực tiếp bằng `root`.

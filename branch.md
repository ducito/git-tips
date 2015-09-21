# Git branch và mô hình pull request

## Tình huống

Hệ thống đang chạy ổn định, khách hàng muốn thử nghiệm tính năng mới foo, bạn bắt tay vào code tính năng mới này. Một ngày đẹp trời nọ, khi bạn đang code nửa chừng tính năng foo, khách hàng yêu cầu: công ty chúng tôi quyết định tính năng khác nữa bar mới là mục tiêu cần kíp nhất lúc này, vậy bạn hãy tạm dừng tính năng foo và chuyển sang làm bar. Khách hàng là thượng đế mà, muốn chửi nó lắm những vẫn nhe răng cười thôi. Bây giờ bạn tính sao? Đang code nửa chừng rồi không lẽ revert về giai đoạn chưa làm, để làm tính năng mới? Hay là copy rồi sao lưu code?
Git cung cấp cho ta giải pháp rẽ nhánh (branch) để giải quyết những trường hợp này (và nhiều trường hợp khó nhằn khác).
Ở phần 1, chúng ra sẽ tìm hiểu về git branch.

## Branch?

Branch là nhánh, khi bạn tách ra một nhánh mới để làm (vd: foo), nhánh mới sẽ thừa hưởng code của nhánh gốc, ngay tại thời điểm bạn tạo nhánh mới. Mọi thay đổi ở nhánh này sẽ không ảnh hưởng gì đến nhánh khác. Ở tình huống đầu bài, bạn chỉ việc quay về nhánh chính (thường là master hoặc development) và tạo nhánh mới bar và phát triển tính năng bar.
Lưu ý Bạn phải quay về nhánh master vì nếu ở nhánh foo thì nhánh bar sẽ chứa luôn cả những đoạn code dang dở của nhánh foo.

### Thao tác với branch

- Trước hết tạo một dự án mới

`$ mkdir demo_branch`

`$ cd demo_branch`

`$ git init`

`$ git remote add gh git@github.com:ducito/demo_pull_request.git`
 
Xin mê: Sao lại là `gh` mà không phải `origin` như tôi vẫn thường thấy/làm?

Cái link git tôi thấy cũng lạ lạ, sao không có HTTPS?

Ở đây mình sử dụng giao thức `ssh`, mời đọc bài viết Tạo `ssh key` và sử dụng `ssh key` trong `git` của mình để hiểu hơn
Khi mới tạo dự án thì chưa có `branch` nào được tạo, nên mình sẽ tạo `branch` gốc, thường thì tên nó sẽ là `master`

- Tạo và chuyển sang `branch` mới:

`$ git checkout -b master`

Tham số `-b` là để tạo `branch` nếu chưa có, nếu có rồi sẽ có lỗi `fatal: A branch named 'foo' already exists.`

- Chuyển sang branch sẵn có

`$ git checkout master`

Tạo một file và add vào index, sau đó commit (lệnh commit sẽ add file vào branch hiện tại)

`$ touch file_in_master`

`$ git add file_in_master`

`$ git commit -m "create first file in master"`
 
Tạo và huyển sang nhánh `foo`, sau đó tạo một file mới, `add` và `commit` nó vào `branch foo`

`$ git checkout -b foo`

`$ touch file_in_foo`

`$ git add file_in_foo`

`$ git commit -m "Add first file in foo"`

Đến đây, trong thưc mục gốc của mình sẽ có hai file: `file_in_master` và `file_in_foo`
`$ ls`

`f`

Thử chuyển về master xem nào:

`$ git checkout master`

`$ ls`

`file_in_master`
 
Tada, file_in_foo đã “ở lại” foo branch rồi 😀
Lưu ý File và những thay đổi trong file chỉ thực sự thuộc về branch hiện tại khi bạn đã add và commit nó vào branch. Nếu chỉ add không cũng không được. Thử tí nhé:

`$ touch new_file_in_master`

`$ echo "edit in master" >> new_file_in_master`

`$ ls`

`file_in_master      new_file_in_master`

`$ git checkout foo`

`Switched to branch 'foo'`

`$ ls`

`file_in_foo     file_in_master      new_file_in_master`

Bạn có thể thấy là new_file_in_master nó vẫn theo lệnh switch đi sang foo branch. Thực tế là do file này chưa thuộc về bất cứ branch nào cả, nó vẫn ở đó thôi, và mọi branch đều có quyền truy cập và tương tác vào nó (kể cả khi bạn đã add nó vào git index bằng lệnh git add new_file_in_master). Do đó, bạn hoàn toàn có thể chỉnh sửa, add, và commit nó lên foo branch:

`$ echo "edit in foo" >> new_file_in_master `

`$ git add new_file_in_master`

`$ git commit -m "commit new file in master to foo branch, it'll belong to foo branch, you can see it on master branch any more"`

`[foo 96110a9] commit new file in master to foo branch, it'll belong to foo branch, you can see it on master branch any more`

` 1 file changed, 2 insertions(+)`

` create mode 100644 new_file_in_master`

`$ git checkout master`

`Switched to branch 'master'`

`$ ls`

`file_in_master
`


Ố ồ, new_file_in_master đã ở lại bên foo branch rồi, mặc dù nó được tạo ra từ master branch.
Quay lại vấn đề chính nào.

Giải quyết yêu cầu đầu bài

Điểm lại, bây giờ ta đã có hai branch:

`* master với file_in_master`

`* foo với 3 file: file_in_master, file_in_foo và new_file_in_master`

Bây giờ khách hàng yêu cầu làm tính năng bar, thì mình về lại master và tạo bar branch

`$ git checkout master`

`Switched to branch 'master'`

`$ touch file_in_bar`

`$ git add file_in_bar`

`$ git commit -m "develop feature bar"`

`[bar bc4a53b] develop feature bar`

`1 file changed, 0 insertions(+), 0 deletions(-)`

`create mode 100644 file_in_bar`

`$ ls`

`file_in_bar file_in_master`
 
Vậy là bar branch sẽ gồm những tính năng đã có ở master, đồng thời có thêm những tính năng mới của bar, còn foo mình đang phát triển dở thì nó vẫn ở bên kia.
Để làm tiền đề cho bài viết tiếp theo, chúng ta push code ở master lên server:

`$ git push gh -u master`

`Counting objects: 11, done.`

`Delta compression using up to 4 threads.`

`Compressing objects: 100% (7/7), done.`

`Writing objects: 100% (11/11), 975 bytes | 0 bytes/s, done.`

`Total 11 (delta 1), reused 0 (delta 0)`

`To git@github.com:CQBinh/demo_pull_request.git`
`
` * [new branch]      master -> master`

`Branch master set up to track remote branch master from gh.`

Đến đây, bạn đã hiểu về git branch, vậy mô hình pull request là gì, và nó giúp ích được gì cho ta, nó có liên quan gì đến git branch. 
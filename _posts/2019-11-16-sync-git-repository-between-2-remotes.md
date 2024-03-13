---
title: Đồng bộ các nhánh của 2 Git repository thuộc 2 server khác nhau
date: 2019-11-16 19:10:00 +0700
categories: [Devops, Git]
tags: [Devops, Git, Tools, "Scripts"]
pin: false
---

## Đặt vấn đề

Bạn muốn chuyển một repository sang một git server mới, và repo của bạn có quá nhiều nhánh và không thể chuyển thủ công từng nhánh một được.

## Giải pháp

Ta sử dụng một bash script đơn giản để lấy toàn bộ danh sách nhánh từ remote của repo gốc (`origin`), và push nó sang repo mới (`dest`)

### Cú pháp

Trước tiên, để thuận tiện sử dụng cho script, ta hình dung ra cú pháp mà mình sẽ sử dụng nó:

```bash
./sync.sh path-to-directory origin dest
```

Trong đó:

- `origin` và `dest` là 2 remote được gán vào trong repo local mà ta clone về
- `path-to-directory` là đường dẫn tương đối đến thư mục chứa repo local

### Bước 1: Clone repo gốc

Clone repo gốc

```bash
clone git://path-to-your-repo
cd your-repo
git fetch --all
```

### Bước 2: Gán repo đích

Gán URL của repo đích vào repo local mình vừa clone về

```bash
git remote add dest git://path-to-your-remote-dest
```

### Bước 3: Lấy danh sách nhánh

Lấy danh sách tên nhánh từ remote gốc, ta cần cắt phần tên remote trong tên nhánh. Ví dụ `origin/master` thành `master`

```bash
git branch -r | grep -v '\->' | grep 'origin/' | sed "s/origin\///g"
```

### Bước 4: Sync các nhánh

Thực hiện sync tất cả các nhánh, ở đây ta sẽ dùng vòng lặp while

```bash
function sync {
  while read line
  do
    writeHR;
    echo "Checking out $line";
    git checkout $line;
    git pull origin $line;
    git pull dest $line;
    git push dest $line;
  done;
}
```

### Script hoàn chỉnh

Từ các bước ở trên, ta có script hoàn chỉnh như sau:

```bash
#!/bin/bash

function writeHR {
  echo '---------------------'
}

function sync {
  while read line
  do
    writeHR;
    echo "Checking out $line";
    git checkout $line;
    git pull $src $line;
    git pull $dest $line;
    git push $dest $line;
  done;
}

cd $1;
echo "You are currently on `realpath .`"

src="$2";
dest="$3";

echo "Fetching remote: $src";
git fetch $src;
echo "Fetching remote: $dest";
git fetch $dest;

git branch -r | grep -v '\->' | grep $src/ | sed "s/$src\///g" | sync
```

Ở đây ta sử dụng `$src` và `$dest` làm tên 2 remote, 2 biến này sẽ được truyền vào từ câu lệnh gọi script

```bash
./sync.sh path-to-directory src-remote dest-remote
```

Gist: [https://gist.github.com/thanhtunguet/bac5a6b6fc21a6b0ada66cc7fee1e776](https://gist.github.com/thanhtunguet/bac5a6b6fc21a6b0ada66cc7fee1e776)

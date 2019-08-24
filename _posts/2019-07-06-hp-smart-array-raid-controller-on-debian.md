---
layout: post
title: "Sử dụng HP Smart Array Raid Controller trên Debian"
date: 2019-07-06 12:00:00.000000000 +07:00
author: "Long Tom"
type: post
published: true
status: publish
categories: 
- Tutorials
tags:
- ssacli
- hpe
- raid
excerpt: "Máy chủ của tôi vừa được cắm thêm ổ cứng. Tiếp theo tôi sẽ phải tạo RAID và mount các ổ cứng này vào OS để sử dụng. Đơn giản nhất là khởi động lại máy chủ rồi sử dụng công cụ HP Smart Array. Tuy nhiên việc này sẽ ảnh hưởng đến các dịch vụ đang chạy và theo lẽ thường phải thực hiện vào ban đêm. Tôi không thích làm đêm nên tôi đã sử dụng HP Smart Storage Administrator CLI để tạo RAID cho các ổ cứng mới mà không phải khởi động tại server"
---

## Giới thiệu

Máy chủ của tôi vừa được cắm thêm ổ cứng. Tiếp theo tôi sẽ phải tạo RAID và mount các ổ cứng này vào OS để sử dụng. Đơn giản nhất là khởi động lại máy chủ rồi sử dụng công cụ HP Smart Array. Tuy nhiên việc này sẽ ảnh hưởng đến các dịch vụ đang chạy và theo lẽ thường phải thực hiện vào ban đêm. Tôi không thích làm đêm nên tôi đã sử dụng HP Smart Storage Administrator CLI (ssacli) để tạo RAID cho các ổ cứng mới mà không phải khởi động tại server

## Cài đặt

Để cài đặt ```ssacli``` tôi chạy đoạn lệnh sau:
> Đây là đoạn lệnh cài đặt trên Debian 9, với phiên bản khác của hệ điều hành, bạn sẽ phải sửa lại đôi chút

```bash
curl http://downloads.linux.hpe.com/SDR/hpPublicKey1024.pub | apt-key add -
curl http://downloads.linux.hpe.com/SDR/hpPublicKey2048.pub | apt-key add -
curl http://downloads.linux.hpe.com/SDR/hpPublicKey2048_key1.pub | apt-key add -
curl http://downloads.linux.hpe.com/SDR/hpePublicKey2048_key1.pub | apt-key add -
echo -e "deb http://downloads.linux.hpe.com/SDR/repo/mcp/ stretch/current non-free" > /etc/apt/sources.list.d/hpe.list
apt-get update && apt-get install ssacli
```

Bây giờ, hãy bắt đầu sử dụng bằng cách gõ lệnh ```ssacli```. Trên màn hình sẽ hiện kết quả tương tự như sau:

```bash
HP Smart Storage Administrator CLI 3.10.3.0
    Detecting Controllers... Done.
    Type “help” for a list of supported commands.
    Type “exit” to close the console.
    =>
```

## Tạo RAID

Trước tiên, tôi sẽ chạy lệnh để hiển thị trạng thái các ổ cứng đã được cắm vào Server. Câu lệnh chạy và kết quả tương tự như dưới đây:

```bash
=> ctrl all show config

Smart Array P410i in Slot 0 (Embedded)    (sn: 50014380101D61C0)

    array A (SAS, Unused Space: 0  MB)

        logicaldrive 1 (136.7 GB, RAID 1, OK)

        physicaldrive 1I:1:1 (port 1I:box 1:bay 1, SAS, 146 GB, OK)
        physicaldrive 1I:1:2 (port 1I:box 1:bay 2, SAS, 146 GB, OK)

    unassigned
        physicaldrive 1I:1:3 (port 1I:box 1:bay 3, SAS, 300 GB, OK)
        physicaldrive 1I:1:4 (port 1I:box 1:bay 4, SAS, 300 GB, OK)
        physicaldrive 2I:1:5 (port 2I:box 1:bay 5, SAS, 300 GB, OK)
        physicaldrive 2I:1:6 (port 2I:box 1:bay 6, SAS, 300 GB, OK)
```

Theo kết quả trên, tôi đang có 4 ổ chưa được tạo RAID tại Slot 0, array A. Giờ tôi sẽ tạo RAID 0 cho 4 ổ này.

```bash
=> ctrl slot=0 create type=ld drives=1I:1:3,1I:1:4,2I:1:5,2I:1:6 raid=0
```

Mở rộng thêm 1 chút, nếu muốn tạo RAID 5 thì bạn chạy lệnh sau. =))

```bash
=> ctrl slot=0 create type=ld drives=1I:1:3,1I:1:4,2I:1:5,2I:1:6 raid=5
```

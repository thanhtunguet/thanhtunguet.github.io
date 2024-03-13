---
title: Giải quyết vấn đề Dynamic IP với Cloudflare API và Telegram Bot
date: 2023-03-26 19:10:00 +0700
categories: [Devops, Cloudflare]
tags: [Devops, DNS, Cloudflare, "Scripts"]
pin: false
---

## Đặt vấn đề

Bạn là sinh viên cần làm bài tập lớn mà không muốn mất quá nhiều tiền để mua cloud?
Bạn có nhu cầu thiết lập một web server tại nhà và tận dụng chiếc máy tính cũ của mình làm máy chủ?
Bạn có camera IP hay các thiết bị IoT khác và muốn truy cập được chúng từ xa?
Và còn vô vàn các nhu cầu khác nữa.

Rõ ràng, ngay trong nhà bạn, bạn đã có một đường mạng Internet mà bạn vẫn dùng hàng ngày, chiếc máy tính để bàn hoặc những chiếc laptop đã rất cũ mà bạn không còn dùng đến nữa. Liệu ta có thể tận dụng chúng để thiết lập một máy chủ để truy cập được từ ngoài Internet hay không? Việc đó sẽ gặp những vấn đề gì? Trong bài này, mình sẽ giúp bạn tìm hiểu và hướng dẫn bạn cách tận dụng những thứ "sẵn có" để phục vụ nhu cầu của mình nhé. Đồng thời, mình cũng hi vọng bạn có thêm một chút kiến thức về mạng nếu chưa biết.

Ngoài ra, bài này hướng dẫn cách tận dụng đường mạng internet gia đình (có địa chỉ IP động), nên nếu bạn đã có sẵn địa chỉ IP tĩnh thì cứ mạnh dạn bỏ qua bài viết này nhé.

## Chuẩn bị

Để bắt đầu, bạn cần chuẩn bị một số thứ sau đây:

- Một đường mạng ổn định và bạn có quyền truy cập vào router tổng để có thể cấu hình mở port kết nối từ bên ngoài. Điều này là khá dễ dàng với các bạn đang ở chính nhà của mình nhưng sẽ hơi khó khăn với các bạn ở trọ vì thường thì các nhà trọ sẽ lắp tối thiểu 2 lớp mạng và bạn không có quyền truy cập vào router tổng.
- Một chiếc máy tính, tất nhiên rồi.
- Một tên miền. Bạn có thể đăng ký miễn phí tại [https://www.freenom.com/](https://www.freenom.com/) hoặc mua nếu có điều kiện.
- Các chiếc máy tính / thiết bị internet khác mà bạn cần.

Lưu ý:
- Một số ISP, ví dụ FPT Telecom, sẽ mặc định không mở port và bạn cần gọi điện cho tổng đài để yêu cầu họ làm việc đó cho mình.

## Vấn đề IP động

Thông thường, các gói internet của gia đình chỉ phục vụ mục đích truy cập Internet chứ không được sử dụng để bên ngoài truy cập vào nên chúng đều được cấu hình IP động. Việc thiết lập IP động có mục đích lớn nhất là giúp tiết kiệm chi phí vì IP là một tài nguyên hữu hạn. Bạn có thể đọc thêm [tại đây](https://wiki.matbao.net/ip-la-gi-tong-hop-moi-kien-thuc-can-biet-ve-dia-chi-ip/).

Và, đó chính là vấn đề. Vì địa chỉ IP là động, nên nó sẽ thường xuyên thay đổi thậm chí là trong ngày, dẫn đến việc bạn đã kiểm tra và ghi chú lại địa chỉ IP của nhà mình, nhưng vừa chạy ra ngoài cái là nó đã đổi, và bạn không thể truy cập được về mạng nhà mình nữa.

## Giải pháp

Giải pháp cho việc này khá đơn giản. Có 2 cách để khắc phục:

### Mua địa chỉ IP tĩnh

Nhanh, gọn, tiết kiệm thời gian, nhưng mất thêm gấp đôi tiền. :)) Nếu bạn làm theo cách này, bạn không cần đọc tiếp phần sau. Nhưng vì chúng ta muốn tận dụng những gì sẵn có, không muốn mất quá nhiều tiền, và muốn hiểu thêm một chút về mạng, nên ta đi đến giải pháp thứ hai.

### Gắn tên miền cố định và cập nhật IP cho tên miền mỗi khi thay đổi

Sử dụng một script liên tục kiểm tra địa chỉ IP và cập nhật lại IP và thông báo cho mình mỗi khi cập nhật. Ở đây mình sử dụng Cloudflare và Telegram.

Khi đó, mình có thể truy cập về mạng nhà thông qua tên miền mà mình đã thiết lập. Bot sẽ tự động kiểm tra IP nếu thay đổi và cập nhật lại vào DNS của Cloudflare để đảm bảo tên miền luôn trỏ đúng về IP mạng nhà mình.

### Cloudflare là gì?

Cloudflare là một công ty cung cấp dịch vụ bảo mật web và tăng tốc độ truy cập trang web. Nó được thành lập vào năm 2009 và hiện là một trong những công ty hàng đầu trong lĩnh vực này.

Cloudflare cung cấp nhiều dịch vụ, bao gồm: bảo vệ chống tấn công DDoS, tường lửa ứng dụng web, bộ lọc spam và phần mềm chống virus, tăng tốc độ trang web bằng cách phân phối nội dung (CDN), và cải thiện bảo mật DNS. Ngoài ra, Cloudflare còn cung cấp các công cụ quản lý và theo dõi trang web, giúp người dùng dễ dàng quản lý và điều khiển trang web của họ.

Ở đây, một trong những điểm hữu ích nhất của Cloudflare là khả năng cập nhật DNS cực nhanh, nhanh hơn hầu hết các DNS trong nước và điều này giúp ta có thể cập nhật nhanh chóng địa chỉ IP của mình cho tên miền.

### Telegram

Telegram thì là một ứng dụng nhắn tin mà hầu hết mọi người đều biết, nó nhanh, ổn định và hỗ trợ thiết lập bot khá dễ dàng. Ta có thể thiết lập một con bot để gửi tin nhắn đến điện thoại của mình, thông báo mỗi khi địa chỉ IP ở mạng nhà mình thay đổi.

## Cách làm chi tiết

Dưới đây là các bước chi tiết thực hiện:

### Tạo Cloudflare API Token

Để tạo Cloudflare API Token với quyền chỉnh sửa DNS, bạn làm theo các bước sau:

Bước 1: Đăng nhập vào tài khoản Cloudflare của bạn.

Bước 2: Chọn "API Tokens" từ menu bên trái.

Bước 3: Nhấp vào "Create Token".

Bước 4: Chọn "Edit zone DNS" ở mục "Permission".

Bước 5: Điền tên cho API Token.

Bước 6: Để cho API Token của bạn hoạt động được trên toàn bộ tên miền của bạn, chọn "All zones" trong mục "Zone Resources". Nếu bạn chỉ muốn API Token hoạt động trên một số tên miền cụ thể, hãy chọn "Selected zones" và chọn các tên miền mà bạn muốn.

Bước 7: Nhấp vào nút "Create".

Bước 8: API Token của bạn sẽ được tạo ra. Lưu trữ API Token này ở một nơi an toàn vì nó sẽ không hiển thị lại.

Sau khi tạo API Token, bạn có thể sử dụng nó để cấu hình DNS cho script tự động cập nhật địa chỉ IP của mình.

### Tạo tên miền phụ và trỏ về IP hiện tại

Để kiểm tra địa chỉ IP hiện tại của nhà mình, thực hiện lệnh sau:

```bash
echo $(curl -s https://checkip.amazonaws.com)
```

Tạo một bản ghi `A` trong Cloudflare trỏ đến địa chỉ IP hiện tại.

### Tìm hiểu về ZONE_ID và RECORD_ID trong Cloudflare DNS

Zone ID và Record ID là hai thông số được sử dụng trong hệ thống DNS của Cloudflare để định danh các tên miền và bản ghi DNS.

Zone ID là một chuỗi ký tự duy nhất được sử dụng để định danh một tên miền trong hệ thống Cloudflare DNS. Mỗi tên miền trong tài khoản Cloudflare của bạn sẽ có một Zone ID riêng. Zone ID được sử dụng để xác định phạm vi của các hoạt động DNS được thực hiện trên tên miền đó, chẳng hạn như thêm, xóa hoặc chỉnh sửa các bản ghi DNS.

Record ID là một chuỗi ký tự duy nhất được sử dụng để định danh một bản ghi DNS cụ thể trong hệ thống Cloudflare DNS. Mỗi bản ghi DNS cho một tên miền sẽ có một Record ID riêng. Record ID được sử dụng để xác định các bản ghi DNS cần được thay đổi, ví dụ như cập nhật giá trị của một bản ghi A hoặc thêm một bản ghi mới.

Việc lấy được Zone ID và Record ID là rất quan trọng trong việc quản lý DNS của tên miền trên hệ thống Cloudflare. Chúng được sử dụng để thực hiện các hoạt động như cập nhật bản ghi DNS, tạo ra các bộ lọc và báo cáo trên hệ thống Cloudflare DNS

### Lấy ZONE_ID và RECORD_ID của bản ghi DNS

Để lấy Zone ID và Record ID trong Cloudflare DNS, bạn có thể làm theo các bước sau:

Bước 1: Đăng nhập vào tài khoản Cloudflare của bạn.

Bước 2: Chọn tên miền mà bạn muốn lấy Zone ID hoặc Record ID.

Bước 3: Nhấp vào tab "DNS".

Bước 4: Tại đây, bạn sẽ thấy tất cả các bản ghi DNS cho tên miền của mình.

Bước 5: Để lấy Zone ID, bạn có thể xem ở phía trên cùng của trang "Overview". Nó sẽ được hiển thị bên cạnh tên miền của bạn.

Bước 6: Để lấy Record ID, bạn có thể nhấp vào biểu tượng "Edit" bên phải của bản ghi DNS mà bạn muốn lấy Record ID. Một hộp thoại sẽ xuất hiện, và Record ID sẽ được hiển thị ở phần cuối của URL trong trình duyệt của bạn. Ví dụ: https://dash.cloudflare.com/xxxxxxxxxxxxxxxxxxxxxxxxxx/edit/xxxxxxxxxxxxxxxxxxxxxxxxxx.

### Tạo Bot cho Telegram

Bước 1: Tìm kiếm tài khoản "BotFather" trong Telegram và bắt đầu cuộc trò chuyện với nó.

Bước 2: Gửi tin nhắn "/newbot" để bắt đầu tạo Bot mới.

Bước 3: Nhập tên cho Bot của bạn.

Bước 4: Sau đó, nhập tên người dùng cho Bot của bạn. Lưu ý rằng tên người dùng phải kết thúc bằng từ "bot", ví dụ: "mytelegrambot_bot".

Bước 5: Nếu mọi thứ diễn ra thuận lợi, BotFather sẽ gửi lại cho bạn một token truy cập. Hãy lưu token này, vì bạn sẽ cần nó để kết nối với Bot của mình.

Bước 6: Bây giờ, bạn đã tạo thành công một Bot trong Telegram. Bạn có thể sử dụng token truy cập của mình để kết nối với Bot của mình thông qua API của Telegram. Bạn có thể sử dụng các thư viện lập trình như Telebot, pyTelegramBotAPI hoặc python-telegram-bot để lập trình Bot của mình.

### Lấy chat ID của tài khoản Telegram

Chat ID của một tài khoản Telegram là một số nguyên duy nhất được sử dụng để định danh cho tài khoản đó. Chat ID được sử dụng để tương tác với các Bot hoặc API Telegram, chẳng hạn như gửi tin nhắn đến một tài khoản cụ thể, lấy thông tin tài khoản hoặc thực hiện các hoạt động khác.

Để lấy chat ID của tài khoản hiện tại trong Telegram, bạn có thể làm theo các bước sau:

Bước 1: Mở ứng dụng Telegram và tìm kiếm tài khoản "@userinfobot".

Bước 2: Bắt đầu một cuộc trò chuyện với tài khoản "@userinfobot".

Bước 3: Gửi tin nhắn "/start" để bắt đầu sử dụng bot.

Bước 4: Bot sẽ gửi lại cho bạn một tin nhắn chứa thông tin về tài khoản của bạn, bao gồm chat ID.

Bước 5: Sao chép chat ID để sử dụng cho mục đích lập trình hoặc tương tác với Telegram API.

### Viết script đầy đủ

Bây giờ ta đã thiết lập xong bản ghi DNS và bot cho Telegram, ta sẽ viết một BASH script hoàn chỉnh để thực hiện các công việc sau:

- Kiểm tra địa chỉ IP hiện tại của mạng gia đình
- Kiểm tra địa chỉ IP trên bản ghi DNS của Cloudflare
- So sánh 2 IP trên. Nếu khác nhau nghĩa là IP của mạng đã thay đổi. Trong trường hợp này ta sẽ gửi thông báo bằng bot đến telegram và thực hiện cập nhật lại địa chỉ IP trên Cloudflare.

```bash
#!/bin/bash

# Cấu hình Cloudflare
CF_API_KEY="CLOUFLARE_API_KEY"
CF_EMAIL="your_email@example.com"
ZONE_ID="ZONE_ID"
RECORD_ID="RECORD_ID"
# Cấu hình Telegram Bot
BOT_TOKEN="Bot_token"
CHAT_ID="Your chat id"
# DNS
YOUR_DOMAIN="*.example.com"
DNS_SERVER="8.8.8.8"

# Function có nhiệm vụ gửi tin nhắn từ bot telegram đến tài khoản chỉ định
function sendChat() {
    curl -s -X POST "https://api.telegram.org/bot${BOT_TOKEN}/sendMessage" \
         -d "chat_id=${CHAT_ID}" \
        -d "text=$1"
}
# Câu lệnh Curl cho Cloudflare
function cloudflare() {
  if [ -z "$CF_EMAIL" ]; then
      curl -sSL \
      -H "Accept: application/json" \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer $CF_API_KEY" \
      "$@"
  else
      curl -sSL \
      -H "Accept: application/json" \
      -H "Content-Type: application/json" \
      -H "X-Auth-Email: $CF_EMAIL" \
      -H "X-Auth-Key: $CF_API_KEY" \
      "$@"
  fi
}

# Lấy địa chỉ IP công khai hiện tại của máy tính
PUBLIC_IP=$(curl -s https://checkip.amazonaws.com)
# Lấy địa chỉ IP hiện tại của DNS
CURRENT_IP=$(nslookup "${YOUR_DOMAIN}" ${DNS_SERVER} | awk '/^Address: / { print $2 }')

# So sánh 2 IP
if [ "$CURRENT_IP" != "$PUBLIC_IP" ]; then
    # Nếu khác nhau, thông báo và cập nhật IP
    sendChat "Current public IP is: ${PUBLIC_IP} was different from current Cloudflare setting: ${CURRENT_IP} and has been updated"
    cloudflare -X PUT \
      "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records/${RECORD_ID}" \
      --data "{\"type\":\"A\",\"name\":\"*\",\"content\":\"${PUBLIC_IP}\",\"ttl\":1,\"proxied\":false}"
else
    # Nếu không khác, bỏ qua
    echo "Current public IP is up to date"
    # Do nothing
fi
```

Lưu ý rằng bạn cần thay các giá trị `ZONE_ID`, `RECORD_ID`, `CF_API_KEY`, `CF_EMAIL`, `BOT_TOKEN` và `CHAT_ID` tương ứng với các giá trị tạo ra từ các bước trên.

### Thiết lập cronjob

Ta đã có script đầy đủ, bây giờ thiết lập cronjob để thực hiện script trên mỗi phút một lần, giúp DNS luôn được cập nhật nhanh nhất có thể.

Bước 1: Lưu script trên thành câu lệnh `/usr/local/bin/cloudflare-check` và cấp quyền thực thi:

```bash
chmod a+x /usr/local/bin/cloudflare-check
```

Bước 2, thiết lập crontab với lệnh `crontab -e`

```bash
*/5 * * * * /usr/local/bin/cloudflare-check >/dev/null 2>&1
```

## Kết luận

Với Cloudflare DNS và Telegram bot, ta có thể viết một script (bot) để kiểm tra địa chỉ IP và cập nhật nó vào một tên miền cố định, giúp ta có thể truy cập được vào mạng gia đình / nội bộ từ bên ngoài một cách dễ dàng.

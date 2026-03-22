# 🚀 Railway 部署指南 — 小Man 療癒急救室

## 需要上傳的檔案（共4個）

```
healing-emergency-room.html
server.py
requirements.txt
Procfile
```

---

## 部署步驟

### 第一步：建立 GitHub 帳號（已有可跳過）
前往 https://github.com → 免費註冊

---

### 第二步：建立 GitHub Repository（倉庫）

1. 登入 GitHub 後，點右上角 **＋** → **New repository**
2. Repository name 輸入：`healing-room`（或任何名字）
3. 選 **Public** 或 **Private** 均可
4. 點 **Create repository**

---

### 第三步：上傳網站檔案到 GitHub

**方法一：網頁上傳（最簡單）**

1. 進入你剛建立的 repository
2. 點 **uploading an existing file**（或 **Add file** → **Upload files**）
3. 把以下 4 個檔案全部拖入上傳：
   - `healing-emergency-room.html`
   - `server.py`
   - `requirements.txt`
   - `Procfile`
4. 點下方 **Commit changes** → **Commit changes**

---

### 第四步：建立 Railway 帳號

1. 前往 https://railway.app
2. 點 **Start a New Project**
3. 選 **Login with GitHub**（用 GitHub 帳號登入最方便）

---

### 第五步：部署到 Railway

1. 登入後點 **New Project**
2. 選 **Deploy from GitHub repo**
3. 選擇你剛建立的 `healing-room` repository
4. Railway 會自動偵測這是 Python 專案並開始部署

---

### 第六步：等待部署完成

部署大約需要 **2-3 分鐘**。你會看到：
```
Build Successful ✓
Deploy Successful ✓
```

---

### 第七步：取得你的網站網址

1. 部署完成後，點選你的 Service
2. 在 **Settings** 分頁，找到 **Domains**
3. 點 **Generate Domain** → Railway 會給你一個網址，例如：
   ```
   https://healing-room-production.up.railway.app
   ```
4. 點擊這個網址，你的小Man 療癒急救室就上線了！🎉

---

## 如果想用自己的域名（例如 wclawicc.org.hk 的子域）

在 Railway 的 **Settings → Domains** 中：
1. 點 **Add Custom Domain**
2. 輸入你想用的子域名，例如：`healing.wclawicc.org.hk`
3. Railway 會顯示一個 CNAME 記錄，例如：
   ```
   CNAME  healing  →  xxxx.railway.app
   ```
4. 到你的域名服務商（Namecheap / GoDaddy / Cloudflare 等）
5. 在 DNS 設定中添加這個 CNAME 記錄
6. 等待 5-30 分鐘生效

---

## 費用說明

| 方案 | 費用 | 適合 |
|------|------|------|
| Hobby（免費試用） | $0/月，有使用限制 | 測試 |
| Hobby（付費） | $5/月 | 正式使用 ✅ |

免費版每月有 500 小時限制，若有穩定訪客建議選 $5/月方案。

---

## 更新網站內容

日後如果想更新網站（例如改文字、加故事），只需：
1. 在 GitHub 上更新檔案
2. Railway 會自動偵測並重新部署（約 1-2 分鐘）

---

## 常見問題

**Q：部署失敗怎麼辦？**
在 Railway 點 **View Logs** 查看錯誤訊息，把截圖發給我幫你看。

**Q：小Man 沒有回應？**
確認 Z.ai API 金鑰仍然有效。可以在 Railway 的 **Variables** 分頁設定環境變數 `ZAI_API_KEY`。

**Q：Railway 免費版夠用嗎？**
初期測試足夠。若有定期用戶，建議升級至 $5/月方案。

---

*有任何問題，截圖發給我，我幫你解決！* 💜

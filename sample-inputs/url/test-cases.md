# URL Input Samples: Cek Loker Link Assessment

Use the short text prompts below to test the VirusTotal API integration and URL analysis capabilities of the Cek Loker agent. Copy the text inside the blockquotes, which includes both the company name and the URL, and paste it into the chat interface.

## 1. Low Risk Scenario (Target Score: 0 | Range: < 20)

**Evidence Context:** The company is officially registered in the Notion database. The provided URL is a reputable official domain, which will return a safe status from VirusTotal.

**Prompt to Paste:**

> Info lowongan PT Mitra Karya Teknologi dapat diakses melalui tautan resmi ini: https://karir.kemnaker.go.id/

---

## 2. Medium Risk Scenario (Target Score: 45 | Range: 20 - 49)

Evidence Context: The company is unregistered (+25). The prompt includes a free email address (gmail) triggering the unofficial contact penalty (+20). The dummy URL will return a neutral/404 state from VirusTotal, adding 0 points. Total score is exactly 45.

**Prompt to Paste:**

> Cek loker admin dari PT Makmur Sentosa Abadi. Berkas lamaran dikirim ke hrd.makmursentosa@gmail.com dan pelamar diminta mengisi data awal di tautan berikut: https://makmursentosa.example.com/pendaftaran

---

## 3. High Risk Scenario (Target Score: 75 | Range: 50+)

**Evidence Context:** Unregistered entity (+25), upfront deposit request (+30), and the use of a free blog domain (+20). This totals 75 points, successfully keeping it in the High Risk category and triggering the automated Google Docs and Gmail draft creation. *(Note: Even if VirusTotal cannot scan the dummy blog domain, the AI will handle the error gracefully based on the system rules).*

**Prompt to Paste:**

> Ada tawaran kerja ke Korea dari PT Mulia Berkat Bersama. Syaratnya harus bayar deposit 500 ribu rupiah dulu ke rekening admin. Pendaftarannya lewat link ini: http://daftar-kerja-korea-vip.blogspot.com/
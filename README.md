# dur ki baten — Tunnel Password Page और Local Tunnel Setup

यह README दो कामों के लिए निर्देश देता है:
1. एक सरल HTML पेज (tunnel-protect.html) जो उपयोगकर्ता से "Tunnel Password" लेता है और आपके टनल URL खोलने में मदद करता है।  
2. वैकल्पिक रूप से एक छोटा Node.js Express सर्वर जो आपकी वेबसाइट को HTTP Basic Auth (username/password) से सुरक्षित कर देता है और उसे ngrok / localtunnel से सार्वजनिक रूप से एक्सपोज़ कर सकता है।

यह समाधान उन स्थितियों के लिए उपयोगी है जहाँ आपने लोकल फ़ोल्डर/वेब ऐप को टनल (ngrok/localtunnel) से शेयर किया है और एक्सेस करने पर एक पासवर्ड चाहिये ताकि केवल वालिद उपयोगकर्ता ही साइट देख सकें।

सामान्य सुरक्षा नोट
- यह तरीका demo/temporary के लिए है। Basic Auth credentials केवल username/password हैं — इसलिए हमेशा HTTPS टनल (ngrok/localtunnel का HTTPS) उपयोग करें ताकि credentials नेटवर्क पर encrypted रहें।
- Production के लिए बेहतर authentication, HTTPS, और proper server-side user management आवश्यक है।

फाइलें (प्रोजेक्ट में)
- tunnel-protect.html — यूज़र‑फ्रेंडली पेज: पासवर्ड दर्ज करने, कॉपी करने और टनल खोलने के लिए।
- server.js — (optional) Express static server + HTTP Basic Auth (env‑based credentials) — public टनल के साथ उपयोग के लिए।
- package.json — (optional) Node dependencies और start script।
- .env.example — example environment variables (TUNNEL_USER, TUNNEL_PASS, PORT)।
- public/ — आपकी वेबसाइट की फ़ाइलें (index.html, css/, js/, assets/) — यदि आप server.js route कर रहे हैं।

1) सिर्फ़ HTML पेज का उपयोग (सबसे सरल)
- यदि आप केवल विज़िटर को पासवर्ड देने का UI चाहते हैं, तो बस `tunnel-protect.html` फाइल खोलें:
  - फाइल खोलने के बाद `<script>` भाग में `TARGET_TUNNEL_URL` को अपने टनल (ngrok/localtunnel) के HTTPS URL से बदलें।
    उदाहरण:
    ```js
    const TARGET_TUNNEL_URL = "https://abcd-1234.ngrok.io";
    ```
  - उपयोगकर्ता पासवर्ड दर्ज करके "Copy" से क्लिपबोर्ड में कॉपी कर सकता है और "Open Tunnel" दबाकर टनल पेज खोलेगा। ब्राउज़र पर Basic Auth prompt आएगा — वहाँ पासवर्ड पेस्ट कर दें।

2) Express + Basic Auth सर्वर (local server with password protection)
- यह तरीका तब उपयोगी है जब आप चाहते हैं कि टनल पर पहुँचने वाले हर अनुरोध पर क्लाइंट‑साइड नहीं बल्कि सर्वर‑साइड authentication लागू हो। ngrok/localtunnel सिर्फ़ आपका लोकल पोर्ट एक्सपोज़ करेंगे; इस सर्वर के साथ credentials सर्वर पर validate होंगे।

फोल्डर structure (उदाहरण)
```
project-root/
│
├─ public/                # आपकी वेब साइट (index.html, css/, js/, assets/)
├─ server.js              # express + basic auth (optional)
├─ package.json
├─ .env.example
└─ tunnel-protect.html    # UI फ़ाइल (optional)
```

server.js (सारांश)
- Express static server serves files from `./public`
- Uses `express-basic-auth` to challenge incoming requests
- Credentials read from `.env` (TUNNEL_USER, TUNNEL_PASS)
- पोर्ट default `8080` (या .env PORT)

कमांड्स (Express method)
1. Node प्रोजेक्ट तैयार करें (यदि पहले से नहीं है):
   ```bash
   npm install
   ```
2. .env बनाएं:
   - Linux / macOS:
     ```bash
     cp .env.example .env
     ```
   - Windows (PowerShell):
     ```powershell
     copy .env.example .env
     ```
   - .env में वैल्यूज़ एडिट करें:
     ```
     TUNNEL_USER=youruser
     TUNNEL_PASS=yourpass
     PORT=8080
     ```
3. सर्वर स्टार्ट करें:
   ```bash
   npm start
   ```
   (या `node server.js` )

4. अब आपका लोकल सर्वर `http://localhost:8080` पर चल रहा होगा और ब्राउज़र पर खोलने पर username/password मागेगा।

3) Lokally expose करने के लिए ngrok या localtunnel इस्तेमाल करना
A) ngrok (सिफारिश)
- ngrok इंस्टॉल करें: https://ngrok.com/download
- (एक बार) authenticate:
  ```bash
  ngrok authtoken <YOUR_NGROK_AUTH_TOKEN>
  ```
- टनल खोलें (यदि server.js पोर्ट 8080 पर चल रहा है):
  ```bash
  ngrok http 8080
  ```
- ngrok आपको public HTTPS URL देगा (उदा. `https://abcd-1234.ngrok.io`) — इस URL को दूसरों को दें। ब्राउज़र पर जाकर login prompt दिखेगा; credentials वही डालें जो आपने `.env` में रखे थे।

B) localtunnel (कोई signup नहीं)
- सीधे चलाएँ (Node उपलब्ध हो):
  ```bash
  npx localtunnel --port 8080 --subdomain mydurkibaten
  ```
- यह आपको URL देगा: `https://mydurkibaten.loca.lt` (यदि subdomain उपलब्ध हो) — फिर वही credentials माँगेगा।

4) Tunnel‑protect.html का उपयोग कैसे करें
- उद्देश्य: आपके उपयोगकर्ता को स्पष्ट UI देना ताकि वे पासवर्ड copy कर सकें और टनल open कर सकें।
- सेट करें:
  - `tunnel-protect.html` के script section में `TARGET_TUNNEL_URL` बदलें:
    ```js
    const TARGET_TUNNEL_URL = "https://abcd-1234.ngrok.io";
    ```
  - उपयोगकर्ता web पेज खोलकर password डालें → Copy → Open Tunnel → दूसरे tab में टनल URL open होगा → browser prompt पर password paste करें।
- यह पेज पासवर्ड सर्वर पर नहीं भेजता — sessionStorage में short-term रखते हैं (local only).

5) Web2Apk / PWABuilder के लिए ZIP बनाना (Quick)
- अगर आप Web→APK builder को ZIP देना चाहते हैं तो public/ फ़ोल्डर को ZIP करें:
  ```bash
  cd project-root
  zip -r site.zip public
  ```
- या GitHub Pages / Netlify पर host करके URL दें (recommended for PWABuilder).

6) Quick Troubleshooting
- यदि ngrok URL पर visit करने पर सीधे site खुल रही है और credential prompt नहीं दिख रही है → जांचें कि आपने server.js में basic auth लागू किया है और server restart किया है।
- यदि browser auto‑login कर रहा है (cached credentials) → use private/incognito window.
- Microphone / camera के लिए अनुमति: यदि आपकी साइट MediaRecorder आदि इस्तेमाल करती है, ब्राउज़र से runtime permission आएगा; WebView द्वारा एक्सेस करने पर कुछ differences हो सकते हैं।

7) Security सलाह
- strong password चुनें (कम से कम 12+ char)।
- सिर्फ़ trusted users के साथ password शेयर करें।
- यदि आप बार‑बार share कर रहे हैं तो consider per‑user credentials या short‑lived tokens।
- ngrok public URL बदल सकता है (free plan), इसलिए लंबे समय तक same URL के भरोसे मत रहें।

8) यदि आप चाहते हैं कि मैं कुछ कर दूँ
- मैं आपके लिए:
  - `server.js`, `package.json`, `.env` template तैयार कर सकता/सकती हूँ (यदि आपने अभी तक नहीं रखा) — मैंने ऊपर का code भी पहले दिया था।
  - आपकी `tunnel-protect.html` में `TARGET_TUNNEL_URL` भर कर पूरी फ़ाइल दे सकता/सकती हूँ — बस HTTPS टनल URL भेजें।
  - एक छोटा script दे सकता/सकती हूँ जो सब कुछ automate करे: install → .env create → server start → ngrok start → और आपको final public URL दे (बताइए आपका OS)।

---

अगर आप चाहें तो अभी अपनी टनल का HTTPS URL दें और मैं आपके लिए `tunnel-protect.html` की एक ready फाइल बना दूँगा जिसमें TARGET_TUNNEL_URL पहले से भरा होगा — आप बस वह फाइल खोल कर उपयोग कर सकते हैं।

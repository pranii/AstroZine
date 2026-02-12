[README.md](https://github.com/user-attachments/files/25269146/README.md)
# ZodiZine 🌟

**An interactive kiosk installation that transforms birthdates into personalized astrological zines**

ZodiZine bridges the digital and physical by combining three ancient zodiac systems (Chinese, Vedic/Indian, and Celtic) to create unique, foldable zine takeaways. Users input their birthday, and the system generates an 8-page printed zine they can fold and keep.

---

## 📖 Project Overview

### Concept
ZodiZine embodies the principle of **"banality as input, interaction as amplifier, emotion as goal"** — transforming a simple birthday into a meaningful, collectible experience that explores identity through the lens of ancient wisdom traditions.

### Features
- ✨ **Three Zodiac Systems**: Chinese, Indian (Vedic), Celtic
- 🎨 **Retro Gaming Aesthetic**: Press Start 2P font, lime green (#c7d800) color scheme
- 📄 **Physical Output**: 8-page foldable zine on A4 paper
- 👆 **Touch-Optimized**: Full kiosk mode with intuitive touch interface
- 🔊 **Engaging Animations**: Loading screens, typewriter effects with sounds
- 🖨️ **Automated Printing**: Direct integration with CUPS printing system

### Target Audience
People aged 23-35 interested in mystical content but seeking tangible experiences rather than purely digital ones.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────┐
│     React/TypeScript Frontend           │
│     (Touch-optimized UI)                │
│  • Birthday Input                       │
│  • Loading Animations                   │
│  • Goodbye Screen                       │
└──────────────┬──────────────────────────┘
               │
               ↓ HTTP/REST
┌──────────────────────────────────────────┐
│     Python/Flask Backend                 │
│  • Zodiac Calculation                    │
│  • PDF Generation (ReportLab)           │
│  • Print Queue Management                │
└──────────────┬───────────────────────────┘
               │
               ↓ CUPS
┌──────────────────────────────────────────┐
│     Physical Printer                     │
└──────────────────────────────────────────┘
```

---

## 🚀 Quick Start

### Prerequisites
- Raspberry Pi 4 (4GB+ RAM)
- HCIK101VCC 10.1" touchscreen
- CUPS-compatible printer
- Node.js 18+, Python 3.9+

### Installation

```bash
# Clone repository
git clone <repository-url>
cd zodizine

# Install frontend dependencies
cd frontend
npm install
npm run build

# Install backend dependencies
cd ../backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Start services
# Terminal 1:
cd backend
source venv/bin/activate
python app.py

# Terminal 2:
cd frontend
npm run serve
```

### First Run
1. Open browser: `http://localhost:5173`
2. Enter a birthday (e.g., 15/03/1990)
3. Watch the loading animation
4. Collect your printed zine!

For full deployment instructions, see [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md)

---

## 📂 Project Structure

```
zodizine/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── BirthdayInput.tsx
│   │   │   ├── LoadingScreen.tsx
│   │   │   └── GoodbyeScreen.tsx
│   │   ├── services/
│   │   │   └── zodiacService.ts
│   │   ├── styles/
│   │   │   ├── fonts.css
│   │   │   ├── index.css
│   │   │   ├── tailwind.css
│   │   │   └── theme.css
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── package.json
│   └── vite.config.ts
│
├── backend/
│   ├── app.py              # Flask server + PDF generation
│   └── requirements.txt
│
├── docs/
│   ├── ZodiZine_Software_Design.md
│   ├── DEPLOYMENT_GUIDE.md
│   └── Zodiacslist.pdf     # Reference data
│
└── README.md
```

---

## 🎨 Design Philosophy

### Visual Language
- **Color Palette**: Lime green (#c7d800) + beige (#E8DCC4) + black
- **Typography**: Press Start 2P (retro gaming aesthetic)
- **Layout**: Minimal, pixel-art inspired, nostalgic

### Interaction Design
- **Input**: Touch-optimized numeric inputs
- **Feedback**: Nokia-style loading bars, typewriter effects
- **Transitions**: Smooth screen transitions with fade effects
- **Sound**: Animal Crossing-inspired typing sounds

### Print Design
The zine follows a specific 8-page folding pattern:

```
Front side:  [6] [5] [1] [2]
Back side:   [8] [3] [4] [7]

Page contents:
1. Cover
2. Chinese Zodiac
3. Indian Zodiac
4. Celtic Zodiac
5. Synthesis (combined traits)
6. Personalized Message
7. Credits
8. Back Cover
```

---

## 🔧 Technical Stack

### Frontend
- **Framework**: React 18 + TypeScript
- **Build Tool**: Vite
- **Styling**: Tailwind CSS
- **Fonts**: Press Start 2P (Google Fonts)

### Backend
- **Server**: Flask (Python 3.9+)
- **PDF Generation**: ReportLab
- **Printing**: CUPS (via subprocess)

### Hardware
- **Device**: Raspberry Pi 4
- **Display**: HCIK101VCC 10.1" touchscreen (1280x800)
- **Input**: Touchscreen (libinput driver)
- **Output**: Any CUPS-compatible printer

---

## 📊 Zodiac Systems

### Chinese Zodiac (生肖)
- **Based on**: Birth year
- **12 Animals**: Rat, Ox, Tiger, Rabbit, Dragon, Snake, Horse, Goat, Monkey, Rooster, Dog, Pig
- **Elements**: Wood, Fire, Earth, Metal, Water

### Indian/Vedic Zodiac (राशि)
- **Based on**: Birth date (sidereal system)
- **12 Rashis**: Makara, Kumbha, Meena, Mesha, Vrishaba, Mithuna, Karkata, Simha, Kanya, Tula, Vrishchika, Dhanus
- **Elements**: Fire, Earth, Air, Water

### Celtic Tree Zodiac
- **Based on**: Birth date
- **13 Trees**: Birch, Rowan, Ash, Alder, Willow, Hawthorn, Oak, Holly, Hazel, Vine, Ivy, Reed, Elder
- **Archetypes**: The Achiever, The Thinker, etc.

---

## 🧪 Testing

### Manual Testing Checklist
- [ ] Touch input responsive on all fields
- [ ] Invalid dates rejected (e.g., Feb 30)
- [ ] Zodiac calculations accurate across edge cases
- [ ] PDF generates correctly
- [ ] Print output matches expected layout
- [ ] Loading animations smooth
- [ ] Sound effects work
- [ ] Screen transitions fluid
- [ ] Folding pattern correct

### Automated Tests
```bash
# Frontend tests (if implemented)
cd frontend
npm test

# Backend tests (if implemented)
cd backend
source venv/bin/activate
pytest
```

---

## 🐛 Troubleshooting

### Common Issues

**Touch not working?**
- Check `/etc/X11/xorg.conf.d/40-libinput.conf`
- Run `sudo evtest` to verify input events
- Restart X11: `sudo systemctl restart lightdm`

**PDF not generating?**
- Check backend logs: `sudo journalctl -u zodizine-backend -f`
- Verify `/tmp/zodizines` directory exists and is writable
- Test ReportLab: `python -c "from reportlab.pdfgen import canvas"`

**Printer not working?**
- Check CUPS status: `sudo systemctl status cups`
- Verify printer in queue: `lpstat -p`
- Test print: `lp /usr/share/cups/data/testprint`

**Frontend not loading?**
- Check if build exists: `ls frontend/dist`
- Verify service: `sudo systemctl status zodizine-frontend`
- Check port 5173 not in use: `sudo lsof -i :5173`

See [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md) for detailed troubleshooting.

---

## 📈 Future Enhancements

### Phase 2 Features
- [ ] Zodiac symbol illustrations
- [ ] QR codes to digital content
- [ ] Numerology integration
- [ ] Moon phase calculation
- [ ] Multi-language support (German/English)

### Commercial Features
- [ ] Brand customization (white-label)
- [ ] Product recommendation engine
- [ ] Analytics dashboard
- [ ] Lookbook generation for fashion brands

---

## 📝 Academic Context

This project was created for the course **"Inszenierung Interaktiver Medien"** (Staging Interactive Media), exploring:
- Hybrid physical-digital experiences
- Janet Murray's framework of immersive environments
- Wolfgang Sting's staging theory
- Human-Centered Design principles (Auernhammer et al.)

The documentation includes a comprehensive academic paper analyzing the project through these theoretical lenses.

---

## 👥 Team

- **Prejni & Cara**: Concept, Design, Documentation
- **Prani**: Software Development, Technical Implementation

---

## 📄 License

[Add appropriate license]

---

## 🙏 Acknowledgments

- Course instructor and peers from "Inszenierung Interaktiver Medien"
- Ancient wisdom keepers of Chinese, Indian, and Celtic traditions
- ReportLab team for excellent PDF generation library
- Press Start 2P font by CodeMan38


**Transform banality into meaning. Create cosmic connections. Print the stars.** ✨

*ZodiZine - Where ancient wisdom meets modern interaction*

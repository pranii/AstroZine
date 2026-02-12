# ZodiZine Software Design Document

## Project Overview
**ZodiZine** is an interactive kiosk installation that generates personalized astrological zines by combining three ancient zodiac systems (Chinese, Indian/Vedic, and Celtic) based on user birthdate input. The system creates physical printed takeaways that users can fold into collectible zines.

**Target Platform:** Raspberry Pi 4 with HCIK101VCC 10.1" touchscreen  
**Tech Stack:** React + TypeScript, Python (PDF generation), CUPS (printing)

---

## System Architecture

### High-Level Architecture
```
┌─────────────────────────────────────────────────────────┐
│                  User Interface Layer                    │
│            (React/TypeScript Frontend)                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │ Birthday │→ │ Loading  │→ │ Goodbye  │              │
│  │  Input   │  │  Screen  │  │  Screen  │              │
│  └──────────┘  └──────────┘  └──────────┘              │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ↓
┌─────────────────────────────────────────────────────────┐
│              Business Logic Layer                        │
│  ┌─────────────────────────────────────────────┐        │
│  │        Zodiac Calculation Service            │        │
│  │  • Chinese Zodiac Calculator                 │        │
│  │  • Indian Zodiac Calculator                  │        │
│  │  • Celtic Zodiac Calculator                  │        │
│  └─────────────────────────────────────────────┘        │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ↓
┌─────────────────────────────────────────────────────────┐
│              PDF Generation Layer                        │
│            (Python/ReportLab Backend)                    │
│  • Template Engine                                       │
│  • Layout Manager (Zine fold structure)                 │
│  • Content Renderer                                      │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ↓
┌─────────────────────────────────────────────────────────┐
│                 Print Layer (CUPS)                       │
└─────────────────────────────────────────────────────────┘
```

---

## Component Specifications

### 1. Frontend Components

#### 1.1 BirthdayInput Component
**Purpose:** Collect user birthdate via touch-optimized interface

**Current Implementation:**
- Three numeric inputs (Day, Month, Year)
- Form validation (ranges: 1-31, 1-12, 1900-2025)
- Retro gaming aesthetic (Press Start 2P font, lime green #c7d800)
- Touch-optimized with `inputMode="numeric"`

**Enhancements Needed:**
```typescript
interface BirthdayInputProps {
  onSubmit: (date: Date) => void;
}

// Add date validation logic
const validateDate = (day: number, month: number, year: number): boolean => {
  const date = new Date(year, month - 1, day);
  return date.getDate() === day && 
         date.getMonth() === month - 1 && 
         date.getFullYear() === year;
};

// Add error messaging for invalid dates
const [error, setError] = useState<string>('');
```

#### 1.2 LoadingScreen Component
**Purpose:** Provide visual feedback during PDF generation and printing

**Current Implementation:**
- Nokia-style loading bars animation
- "PRINTING..." text
- 5-second duration

**Enhancements Needed:**
- Real-time status updates from print queue
- Error handling for print failures
- Progress tracking (generating → sending to printer → printing)

#### 1.3 GoodbyeScreen Component
**Purpose:** Provide folding instructions with engaging typewriter effect

**Current Implementation:**
- Typewriter animation with Animal Crossing-style sounds
- Fade-out transition to black
- Instructions reference wall diagram

**Enhancements Needed:**
- Optional: Animated folding diagram overlay
- QR code to digital version or website

---

### 2. Zodiac Calculation Services

#### 2.1 Chinese Zodiac Calculator

```typescript
interface ChineseZodiacResult {
  animal: string;
  element: string;
  traits: string[];
  years: number[];
  chineseCharacter: string;
  pinyin: string;
}

const CHINESE_ZODIAC = [
  { animal: 'Rat', character: '鼠', pinyin: 'shǔ', element: 'Water', 
    traits: ['Clever', 'resourceful', 'sharp', 'charming', 'communicative'] },
  { animal: 'Ox', character: '牛', pinyin: 'niú', element: 'Earth',
    traits: ['Steady', 'reliable', 'determined', 'dependable', 'loyal'] },
  { animal: 'Tiger', character: '虎', pinyin: 'hǔ', element: 'Wood',
    traits: ['Brave', 'confident', 'fierce', 'leader', 'dynamic', 'passionate'] },
  { animal: 'Rabbit', character: '兔', pinyin: 'tù', element: 'Wood',
    traits: ['Gentle', 'compassionate', 'kind', 'reliable', 'harmonic', 'calm', 'empathetic'] },
  { animal: 'Dragon', character: '龙', pinyin: 'lóng', element: 'Earth',
    traits: ['Powerful', 'bold', 'creative', 'lucky', 'ambitious', 'driven', 'charismatic', 'leader'] },
  { animal: 'Snake', character: '蛇', pinyin: 'shé', element: 'Fire',
    traits: ['Wise', 'insightful', 'sophisticated', 'thoughtful', 'elegant'] },
  { animal: 'Horse', character: '马', pinyin: 'mǎ', element: 'Fire',
    traits: ['Free-spirited', 'energetic', 'adventurous', 'independent', 'positive'] },
  { animal: 'Goat', character: '羊', pinyin: 'yáng', element: 'Earth',
    traits: ['Gentle', 'creative', 'kind', 'empathetic', 'artistic', 'harmonic'] },
  { animal: 'Monkey', character: '猴', pinyin: 'hóu', element: 'Metal',
    traits: ['Witty', 'clever', 'energetic', 'innovative', 'curious', 'playful', 'adaptable', 'creative'] },
  { animal: 'Rooster', character: '鸡', pinyin: 'jī', element: 'Metal',
    traits: ['Hardworking', 'ambitious', 'straightforward', 'precise', 'honest'] },
  { animal: 'Dog', character: '狗', pinyin: 'gǒu', element: 'Earth',
    traits: ['Loyal', 'honest', 'reliable', 'dependable', 'trustworthy'] },
  { animal: 'Pig', character: '猪', pinyin: 'zhū', element: 'Water',
    traits: ['Kind', 'generous', 'compassionate', 'joyful', 'dependable', 'sincere'] }
];

function calculateChineseZodiac(birthYear: number): ChineseZodiacResult {
  // Chinese New Year typically falls between Jan 21 - Feb 20
  // Simplified: Use year % 12, with 1924 as Rat (index 0)
  const index = (birthYear - 1924) % 12;
  const zodiac = CHINESE_ZODIAC[index < 0 ? index + 12 : index];
  
  return {
    animal: zodiac.animal,
    element: zodiac.element,
    traits: zodiac.traits,
    years: Array.from({length: 8}, (_, i) => 1924 + index + (i * 12)),
    chineseCharacter: zodiac.character,
    pinyin: zodiac.pinyin
  };
}
```

#### 2.2 Indian (Vedic) Zodiac Calculator

```typescript
interface IndianZodiacResult {
  sign: string;
  sanskrit: string;
  transliteration: string;
  symbol: string;
  element: string;
  traits: string[];
  dateRange: { start: string; end: string };
}

const INDIAN_ZODIAC = [
  { sign: 'Makara', sanskrit: 'मकर', transliteration: 'makara', 
    westernSign: 'Capricorn', symbol: 'Crocodile (Sea Goat)', element: 'Earth',
    traits: ['Hard-working'],
    start: { month: 1, day: 14 }, end: { month: 2, day: 11 } },
  { sign: 'Kumbha', sanskrit: 'कुम्भ', transliteration: 'kumbha',
    westernSign: 'Aquarius', symbol: 'Water-bearer', element: 'Air',
    traits: ['Innovative', 'friendly', 'creative', 'artistic', 'communicative'],
    start: { month: 2, day: 12 }, end: { month: 3, day: 12 } },
  { sign: 'Meena', sanskrit: 'मीन', transliteration: 'mīna',
    westernSign: 'Pisces', symbol: 'Fishes', element: 'Water',
    traits: ['Empathetic', 'imaginative', 'creative', 'intelligent', 'sensitive', 'sympathetic'],
    start: { month: 3, day: 13 }, end: { month: 4, day: 13 } },
  { sign: 'Mesha', sanskrit: 'मेष', transliteration: 'meṣa',
    westernSign: 'Aries', symbol: 'Ram', element: 'Fire',
    traits: ['Passionate', 'leader', 'driven'],
    start: { month: 4, day: 14 }, end: { month: 5, day: 14 } },
  { sign: 'Vrishaba', sanskrit: 'वृषभ', transliteration: 'vṛṣabha',
    westernSign: 'Taurus', symbol: 'Bull', element: 'Earth',
    traits: ['Wise', 'passionate', 'creative'],
    start: { month: 5, day: 15 }, end: { month: 6, day: 14 } },
  { sign: 'Mithuna', sanskrit: 'मिथुन', transliteration: 'mithuna',
    westernSign: 'Gemini', symbol: 'Twins', element: 'Air',
    traits: ['Spontaneous', 'adventurous', 'understanding', 'communicative', 'curious'],
    start: { month: 6, day: 15 }, end: { month: 7, day: 14 } },
  { sign: 'Karkata', sanskrit: 'कर्क', transliteration: 'karka',
    westernSign: 'Cancer', symbol: 'Crab', element: 'Water',
    traits: ['Artistic', 'sensitive', 'empathetic'],
    start: { month: 7, day: 15 }, end: { month: 8, day: 14 } },
  { sign: 'Simha', sanskrit: 'सिंह', transliteration: 'siṃha',
    westernSign: 'Leo', symbol: 'Lion', element: 'Fire',
    traits: ['Brave', 'admirable', 'cheerful', 'optimistic'],
    start: { month: 8, day: 15 }, end: { month: 9, day: 15 } },
  { sign: 'Kanya', sanskrit: 'कन्या', transliteration: 'kanyā',
    westernSign: 'Virgo', symbol: 'Virgin girl', element: 'Earth',
    traits: ['Intelligent', 'kind', 'creative', 'driven'],
    start: { month: 9, day: 16 }, end: { month: 10, day: 15 } },
  { sign: 'Tula', sanskrit: 'तुला', transliteration: 'tulā',
    westernSign: 'Libra', symbol: 'Balance', element: 'Air',
    traits: ['Charming', 'inviting', 'romantic'],
    start: { month: 10, day: 16 }, end: { month: 11, day: 14 } },
  { sign: 'Vrishchika', sanskrit: 'वृश्चिक', transliteration: 'vṛścika',
    westernSign: 'Scorpio', symbol: 'Scorpion', element: 'Water',
    traits: ['Compassionate', 'empathetic', 'determined', 'strong-willed'],
    start: { month: 11, day: 15 }, end: { month: 12, day: 14 } },
  { sign: 'Dhanus', sanskrit: 'धनुष', transliteration: 'dhanuṣa',
    westernSign: 'Sagittarius', symbol: 'Bow and arrow', element: 'Fire',
    traits: ['Honest', 'freespirit', 'outspoken', 'kind', 'energetic'],
    start: { month: 12, day: 15 }, end: { month: 1, day: 13 } }
];

function calculateIndianZodiac(birthDate: Date): IndianZodiacResult {
  const month = birthDate.getMonth() + 1;
  const day = birthDate.getDate();
  
  const sign = INDIAN_ZODIAC.find(z => {
    if (z.start.month === z.end.month) {
      return month === z.start.month && day >= z.start.day && day <= z.end.day;
    } else if (z.start.month > z.end.month) {
      // Wraps around year (like Dhanus)
      return (month === z.start.month && day >= z.start.day) ||
             (month === z.end.month && day <= z.end.day);
    } else {
      return (month === z.start.month && day >= z.start.day) ||
             (month > z.start.month && month < z.end.month) ||
             (month === z.end.month && day <= z.end.day);
    }
  });
  
  if (!sign) throw new Error('Invalid date');
  
  return {
    sign: sign.sign,
    sanskrit: sign.sanskrit,
    transliteration: sign.transliteration,
    symbol: sign.symbol,
    element: sign.element,
    traits: sign.traits,
    dateRange: {
      start: `${sign.start.month}/${sign.start.day}`,
      end: `${sign.end.month}/${sign.end.day}`
    }
  };
}
```

#### 2.3 Celtic Zodiac Calculator

```typescript
interface CelticZodiacResult {
  tree: string;
  archetype: string;
  traits: string[];
  dateRange: { start: string; end: string };
}

const CELTIC_ZODIAC = [
  { tree: 'Birch', archetype: 'The Achiever',
    traits: ['Ambitious', 'Loving', 'Courageous'],
    start: { month: 12, day: 24 }, end: { month: 1, day: 20 } },
  { tree: 'Rowan', archetype: 'The Thinker',
    traits: ['Intelligent', 'Highly influential', 'Patient'],
    start: { month: 1, day: 21 }, end: { month: 2, day: 17 } },
  { tree: 'Ash', archetype: 'The Enchanter',
    traits: ['Free-thinking', 'Artistic', 'Imaginative'],
    start: { month: 2, day: 18 }, end: { month: 3, day: 17 } },
  { tree: 'Alder', archetype: 'The Trailblazer',
    traits: ['Romantic', 'Brave', 'Generous'],
    start: { month: 3, day: 18 }, end: { month: 4, day: 14 } },
  { tree: 'Willow', archetype: 'The Observer',
    traits: ['Calm', 'Intuitive', 'Sympathetic'],
    start: { month: 4, day: 15 }, end: { month: 5, day: 12 } },
  { tree: 'Hawthorn', archetype: 'The Illusionist',
    traits: ['Secretive', 'Fun', 'Passionate'],
    start: { month: 5, day: 13 }, end: { month: 6, day: 9 } },
  { tree: 'Oak', archetype: 'The Stabiliser',
    traits: ['Loyal', 'Generous', 'Courageous'],
    start: { month: 6, day: 10 }, end: { month: 7, day: 7 } },
  { tree: 'Holly', archetype: 'The Ruler',
    traits: ['Confident', 'Noble', 'Leader'],
    start: { month: 7, day: 8 }, end: { month: 8, day: 4 } },
  { tree: 'Hazel', archetype: 'The Knower',
    traits: ['Confident', 'Loyal', 'Intelligent'],
    start: { month: 8, day: 5 }, end: { month: 9, day: 1 } },
  { tree: 'Vine', archetype: 'The Equaliser',
    traits: ['Charming', 'Loving', 'Elegant'],
    start: { month: 9, day: 2 }, end: { month: 9, day: 29 } },
  { tree: 'Ivy', archetype: 'The Survivor',
    traits: ['Brave', 'Determined', 'Kind'],
    start: { month: 9, day: 30 }, end: { month: 10, day: 27 } },
  { tree: 'Reed', archetype: 'The Inquisitor',
    traits: ['Compassionate', 'Truthful', 'Curious'],
    start: { month: 10, day: 28 }, end: { month: 11, day: 24 } },
  { tree: 'Elder', archetype: 'The Seeker',
    traits: ['Loyal', 'Ambitious', 'Thoughtful'],
    start: { month: 11, day: 25 }, end: { month: 12, day: 23 } }
];

function calculateCelticZodiac(birthDate: Date): CelticZodiacResult {
  const month = birthDate.getMonth() + 1;
  const day = birthDate.getDate();
  
  const sign = CELTIC_ZODIAC.find(z => {
    if (z.start.month === z.end.month) {
      return month === z.start.month && day >= z.start.day && day <= z.end.day;
    } else if (z.start.month > z.end.month) {
      // Wraps around year (like Birch)
      return (month === z.start.month && day >= z.start.day) ||
             (month === z.end.month && day <= z.end.day);
    } else {
      return (month === z.start.month && day >= z.start.day) ||
             (month > z.start.month && month < z.end.month) ||
             (month === z.end.month && day <= z.end.day);
    }
  });
  
  if (!sign) throw new Error('Invalid date');
  
  return {
    tree: sign.tree,
    archetype: sign.archetype,
    traits: sign.traits,
    dateRange: {
      start: `${sign.start.month}/${sign.start.day}`,
      end: `${sign.end.month}/${sign.end.day}`
    }
  };
}
```

---

### 3. Unified Zodiac Service

```typescript
interface ZodiacProfile {
  birthDate: Date;
  chinese: ChineseZodiacResult;
  indian: IndianZodiacResult;
  celtic: CelticZodiacResult;
}

class ZodiacService {
  static generateProfile(birthDate: Date): ZodiacProfile {
    return {
      birthDate,
      chinese: calculateChineseZodiac(birthDate.getFullYear()),
      indian: calculateIndianZodiac(birthDate),
      celtic: calculateCelticZodiac(birthDate)
    };
  }
  
  static async generateAndPrintZine(birthDate: Date): Promise<void> {
    const profile = this.generateProfile(birthDate);
    
    // Call Python backend to generate PDF
    const response = await fetch('/api/generate-zine', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        birthDate: birthDate.toISOString(),
        chinese: profile.chinese,
        indian: profile.indian,
        celtic: profile.celtic
      })
    });
    
    if (!response.ok) {
      throw new Error('Failed to generate zine');
    }
    
    const { pdfPath, jobId } = await response.json();
    
    // Trigger printing
    await fetch('/api/print', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ pdfPath, jobId })
    });
  }
}
```

---

### 4. PDF Generation Backend (Python)

```python
# zine_generator.py
from reportlab.lib.pagesizes import A4, landscape
from reportlab.pdfgen import canvas
from reportlab.lib.units import mm
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont
from datetime import datetime
import json

class ZineGenerator:
    """
    Generates 8-page zine layout on single A4 sheet
    Follows zine folding pattern from ZineLayout.pdf
    """
    
    def __init__(self):
        self.width, self.height = landscape(A4)
        self.page_width = self.width / 4
        self.page_height = self.height / 2
        
        # Register custom fonts
        # pdfmetrics.registerFont(TTFont('PressStart2P', 'PressStart2P.ttf'))
        
    def create_zine(self, zodiac_data, output_path):
        """
        Creates zine PDF with 8 pages arranged for folding
        
        Layout (landscape A4):
        ┌─────┬─────┬─────┬─────┐
        │  6  │  5  │  1  │  2  │  Front side
        ├─────┼─────┼─────┼─────┤
        │  8  │  3  │  4  │  7  │  Back side
        └─────┴─────┴─────┴─────┘
        """
        c = canvas.Canvas(output_path, pagesize=landscape(A4))
        
        # Page layout mapping (position -> logical page number)
        front_layout = [6, 5, 1, 2]
        back_layout = [8, 3, 4, 7]
        
        # Draw front side
        self._draw_side(c, front_layout, zodiac_data, is_front=True)
        c.showPage()
        
        # Draw back side
        self._draw_side(c, back_layout, zodiac_data, is_front=False)
        
        c.save()
        return output_path
    
    def _draw_side(self, c, layout, zodiac_data, is_front):
        """Draw one side of the zine"""
        for i, page_num in enumerate(layout):
            x_offset = i * self.page_width
            y_offset = 0
            
            # Draw page border (for cutting guides)
            c.setStrokeColorRGB(0.8, 0.8, 0.8)
            c.setLineWidth(0.5)
            c.rect(x_offset, y_offset, self.page_width, self.page_height)
            
            # Draw page content
            self._draw_page_content(c, page_num, x_offset, y_offset, zodiac_data)
    
    def _draw_page_content(self, c, page_num, x, y, data):
        """Draw content for specific page"""
        
        # Lime green background for brand consistency
        c.setFillColorRGB(0.78, 0.85, 0)  # #c7d800
        c.rect(x, y, self.page_width, self.page_height, fill=1)
        
        c.setFillColorRGB(0, 0, 0)  # Black text
        
        if page_num == 1:
            # Cover page
            self._draw_cover(c, x, y, data['birthDate'])
        elif page_num == 2:
            # Chinese Zodiac
            self._draw_chinese_zodiac(c, x, y, data['chinese'])
        elif page_num == 3:
            # Indian Zodiac
            self._draw_indian_zodiac(c, x, y, data['indian'])
        elif page_num == 4:
            # Celtic Zodiac
            self._draw_celtic_zodiac(c, x, y, data['celtic'])
        elif page_num == 5:
            # Combined traits / synthesis
            self._draw_synthesis(c, x, y, data)
        elif page_num == 6:
            # Personalized message
            self._draw_personalization(c, x, y, data)
        elif page_num == 7:
            # Credits / about
            self._draw_credits(c, x, y)
        elif page_num == 8:
            # Back cover / folding instructions
            self._draw_back_cover(c, x, y)
    
    def _draw_cover(self, c, x, y, birth_date):
        """Page 1: Cover"""
        c.setFont("Helvetica-Bold", 24)
        c.drawCentredString(
            x + self.page_width/2, 
            y + self.page_height - 30*mm,
            "ZODIZINE"
        )
        
        c.setFont("Helvetica", 12)
        c.drawCentredString(
            x + self.page_width/2,
            y + self.page_height/2,
            f"Born: {birth_date}"
        )
        
        c.setFont("Helvetica", 8)
        c.drawCentredString(
            x + self.page_width/2,
            y + 10*mm,
            "Your cosmic identity across three traditions"
        )
    
    def _draw_chinese_zodiac(self, c, x, y, chinese):
        """Page 2: Chinese Zodiac"""
        c.setFont("Helvetica-Bold", 16)
        c.drawCentredString(x + self.page_width/2, y + self.page_height - 20*mm,
                           "CHINESE ZODIAC")
        
        c.setFont("Helvetica-Bold", 20)
        c.drawCentredString(x + self.page_width/2, y + self.page_height - 35*mm,
                           f"{chinese['chineseCharacter']} {chinese['animal']}")
        
        c.setFont("Helvetica", 10)
        c.drawCentredString(x + self.page_width/2, y + self.page_height - 45*mm,
                           f"({chinese['pinyin']})")
        
        c.setFont("Helvetica-Bold", 12)
        c.drawCentredString(x + self.page_width/2, y + self.page_height - 60*mm,
                           f"Element: {chinese['element']}")
        
        # Traits
        c.setFont("Helvetica", 9)
        y_pos = y + self.page_height - 75*mm
        c.drawCentredString(x + self.page_width/2, y_pos, "TRAITS:")
        y_pos -= 5*mm
        
        for trait in chinese['traits']:
            c.drawCentredString(x + self.page_width/2, y_pos, trait)
            y_pos -= 4*mm
    
    def _draw_indian_zodiac(self, c, x, y, indian):
        """Page 3: Indian Zodiac"""
        c.setFont("Helvetica-Bold", 16)
        c.drawCentredString(x + self.page_width/2, y + self.page_height - 20*mm,
                           "VEDIC ZODIAC")
        
        c.setFont("Helvetica-Bold", 18)
        c.drawCentredString(x + self.page_width/2, y + self.page_height - 35*mm,
                           indian['sanskrit'])
        
        c.setFont("Helvetica", 12)
        c.drawCentredString(x + self.page_width/2, y + self.page_height - 45*mm,
                           f"{indian['sign']} ({indian['transliteration']})")
        
        c.setFont("Helvetica-Bold", 10)
        c.drawCentredString(x + self.page_width/2, y + self.page_height - 55*mm,
                           f"{indian['symbol']} | {indian['element']}")
        
        # Traits
        c.setFont("Helvetica", 9)
        y_pos = y + self.page_height - 70*mm
        for trait in indian['traits']:
            c.drawCentredString(x + self.page_width/2, y_pos, trait)
            y_pos -= 4*mm
    
    def _draw_celtic_zodiac(self, c, x, y, celtic):
        """Page 4: Celtic Zodiac"""
        c.setFont("Helvetica-Bold", 16)
        c.drawCentredString(x + self.page_width/2, y + self.page_height - 20*mm,
                           "CELTIC TREE ZODIAC")
        
        c.setFont("Helvetica-Bold", 20)
        c.drawCentredString(x + self.page_width/2, y + self.page_height - 40*mm,
                           celtic['tree'])
        
        c.setFont("Helvetica-Oblique", 12)
        c.drawCentredString(x + self.page_width/2, y + self.page_height - 50*mm,
                           celtic['archetype'])
        
        # Traits
        c.setFont("Helvetica", 10)
        y_pos = y + self.page_height - 65*mm
        for trait in celtic['traits']:
            c.drawCentredString(x + self.page_width/2, y_pos, trait)
            y_pos -= 5*mm
    
    def _draw_synthesis(self, c, x, y, data):
        """Page 5: Combined interpretation"""
        c.setFont("Helvetica-Bold", 14)
        c.drawCentredString(x + self.page_width/2, y + self.page_height - 20*mm,
                           "YOUR COSMIC SYNTHESIS")
        
        # Collect all unique traits
        all_traits = set(
            data['chinese']['traits'] +
            data['indian']['traits'] +
            data['celtic']['traits']
        )
        
        c.setFont("Helvetica", 9)
        c.drawCentredString(x + self.page_width/2, y + self.page_height - 35*mm,
                           "Core qualities across all traditions:")
        
        y_pos = y + self.page_height - 45*mm
        for trait in sorted(all_traits)[:8]:  # Top 8 traits
            c.drawCentredString(x + self.page_width/2, y_pos, f"• {trait}")
            y_pos -= 5*mm
    
    def _draw_personalization(self, c, x, y, data):
        """Page 6: Personalized message"""
        c.setFont("Helvetica-Bold", 12)
        c.drawCentredString(x + self.page_width/2, y + self.page_height - 25*mm,
                           "A MESSAGE FOR YOU")
        
        # Generate personalized content based on combination
        message = self._generate_personalized_message(data)
        
        c.setFont("Helvetica", 8)
        y_pos = y + self.page_height - 40*mm
        
        # Text wrapping
        from reportlab.lib import utils
        words = message.split()
        line = ""
        for word in words:
            test_line = line + word + " "
            if c.stringWidth(test_line, "Helvetica", 8) < self.page_width - 10*mm:
                line = test_line
            else:
                c.drawCentredString(x + self.page_width/2, y_pos, line.strip())
                line = word + " "
                y_pos -= 4*mm
        if line:
            c.drawCentredString(x + self.page_width/2, y_pos, line.strip())
    
    def _draw_credits(self, c, x, y):
        """Page 7: Credits"""
        c.setFont("Helvetica", 7)
        credits = [
            "ZODIZINE",
            "",
            "A hybrid experience blending",
            "ancient wisdom with modern interaction",
            "",
            "Created by: Prejniii & Prani",
            "Course: Inszenierung Interaktiver Medien",
            "",
            "Zodiac systems:",
            "Chinese • Vedic • Celtic",
            "",
            f"Generated: {datetime.now().strftime('%Y-%m-%d')}"
        ]
        
        y_pos = y + self.page_height - 25*mm
        for line in credits:
            c.drawCentredString(x + self.page_width/2, y_pos, line)
            y_pos -= 4*mm
    
    def _draw_back_cover(self, c, x, y):
        """Page 8: Back cover"""
        c.setFont("Helvetica-Bold", 10)
        c.drawCentredString(x + self.page_width/2, y + self.page_height/2,
                           "Keep this zine")
        c.setFont("Helvetica", 8)
        c.drawCentredString(x + self.page_width/2, y + self.page_height/2 - 6*mm,
                           "as a reminder of your cosmic identity")
    
    def _generate_personalized_message(self, data):
        """Generate unique message based on zodiac combination"""
        chinese = data['chinese']['animal']
        indian = data['indian']['sign']
        celtic = data['celtic']['tree']
        
        messages = {
            # Example personalized combinations
            'default': f"As a {chinese} in the Chinese tradition, {indian} in Vedic astrology, "
                      f"and {celtic} in Celtic wisdom, you embody a unique blend of energies. "
                      f"Your path combines ancient insights from three powerful traditions."
        }
        
        return messages.get(f"{chinese}_{indian}_{celtic}", messages['default'])


# Flask/FastAPI Backend
from flask import Flask, request, jsonify
import subprocess
import os

app = Flask(__name__)

@app.route('/api/generate-zine', methods=['POST'])
def generate_zine():
    data = request.json
    
    # Generate unique filename
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    output_path = f'/tmp/zine_{timestamp}.pdf'
    
    # Create zine
    generator = ZineGenerator()
    generator.create_zine(data, output_path)
    
    return jsonify({
        'pdfPath': output_path,
        'jobId': timestamp
    })

@app.route('/api/print', methods=['POST'])
def print_zine():
    data = request.json
    pdf_path = data['pdfPath']
    
    # Send to printer via CUPS
    try:
        subprocess.run(['lp', pdf_path], check=True)
        return jsonify({'status': 'success', 'message': 'Print job submitted'})
    except subprocess.CalledProcessError as e:
        return jsonify({'status': 'error', 'message': str(e)}), 500

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

---

## 5. Integration & Data Flow

### Complete User Journey

```
1. User approaches kiosk
   ↓
2. BirthdayInput screen displays
   ↓
3. User enters birthday via touch input
   ↓
4. Frontend validates date
   ↓
5. ZodiacService.generateAndPrintZine() called
   ↓
6. LoadingScreen displays (5 seconds)
   ↓
7. Frontend sends POST to /api/generate-zine with zodiac data
   ↓
8. Python backend generates PDF using ReportLab
   ↓
9. Backend returns PDF path
   ↓
10. Frontend sends POST to /api/print
    ↓
11. Backend sends PDF to CUPS printer
    ↓
12. GoodbyeScreen displays with typewriter animation
    ↓
13. Physical zine prints (user retrieves from printer)
    ↓
14. Screen fades to black → returns to BirthdayInput
```

---

## 6. Technical Requirements

### Hardware Setup
- **Device:** Raspberry Pi 4 (4GB+ RAM recommended)
- **Display:** HCIK101VCC 10.1" touchscreen (1280x800)
- **Printer:** Any CUPS-compatible printer
- **Storage:** 32GB+ microSD card

### Software Dependencies

**Frontend:**
```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "typescript": "^5.0.0"
  },
  "devDependencies": {
    "vite": "^4.0.0",
    "@vitejs/plugin-react": "^3.0.0",
    "tailwindcss": "^3.3.0"
  }
}
```

**Backend:**
```txt
Flask==2.3.0
reportlab==4.0.0
Pillow==10.0.0
python-cups==2.0.1
```

### System Configuration

**Touch Input Configuration (`/etc/X11/xorg.conf.d/40-libinput.conf`):**
```conf
Section "InputClass"
    Identifier "HCIK101VCC Touchscreen"
    MatchProduct "HCIK101VCC"
    MatchDevicePath "/dev/input/event*"
    Driver "libinput"
    Option "TransformationMatrix" "1 0 0 0 1 0 0 0 1"
EndSection
```

**Kiosk Mode (Chromium autostart):**
```bash
# ~/.config/autostart/kiosk.desktop
[Desktop Entry]
Type=Application
Name=ZodiZine Kiosk
Exec=chromium-browser --kiosk --disable-infobars --noerrdialogs http://localhost:5173
```

---

## 7. Deployment Steps

### Initial Setup
```bash
# 1. Install system dependencies
sudo apt update
sudo apt install -y nodejs npm python3-pip cups chromium-browser

# 2. Clone/copy project files
cd /home/pi/zodizine

# 3. Install frontend dependencies
npm install
npm run build

# 4. Install Python dependencies
pip3 install -r requirements.txt

# 5. Configure printer
sudo usermod -a -G lpadmin pi
# Add printer via CUPS web interface (localhost:631)

# 6. Setup autostart
cp kiosk.desktop ~/.config/autostart/

# 7. Configure touch input
sudo cp 40-libinput.conf /etc/X11/xorg.conf.d/
sudo reboot
```

### Running the Application
```bash
# Terminal 1: Start Python backend
cd /home/pi/zodizine/backend
python3 app.py

# Terminal 2: Start frontend dev server (or serve build)
cd /home/pi/zodizine
npm run dev
# OR for production: npx serve -s dist -l 5173
```

### Production Deployment
```bash
# Create systemd services for auto-start

# /etc/systemd/system/zodizine-backend.service
[Unit]
Description=ZodiZine Backend
After=network.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/zodizine/backend
ExecStart=/usr/bin/python3 app.py
Restart=always

[Install]
WantedBy=multi-user.target

# /etc/systemd/system/zodizine-frontend.service
[Unit]
Description=ZodiZine Frontend
After=network.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/zodizine
ExecStart=/usr/bin/npx serve -s dist -l 5173
Restart=always

[Install]
WantedBy=multi-user.target

# Enable services
sudo systemctl enable zodizine-backend
sudo systemctl enable zodizine-frontend
sudo systemctl start zodizine-backend
sudo systemctl start zodizine-frontend
```

---

## 8. Future Enhancements

### Phase 2 Features
1. **Image Integration**
   - Zodiac symbol illustrations
   - Decorative borders
   - QR codes linking to digital content

2. **Advanced Personalization**
   - Numerology integration
   - Moon phase at birth
   - Lucky numbers/colors

3. **Multi-language Support**
   - Interface in German/English
   - Zodiac descriptions in multiple languages

4. **Analytics Dashboard**
   - Usage statistics
   - Popular zodiac combinations
   - Print queue monitoring

5. **Offline Resilience**
   - Local data caching
   - Offline mode indicators
   - Print queue management

### Commercial Features
1. **Brand Customization**
   - White-label PDF templates
   - Custom color schemes
   - Brand logo integration

2. **Lookbook Generation**
   - Product recommendations based on zodiac
   - Style personality matching
   - Seasonal collection alignment

---

## 9. Testing Strategy

### Unit Tests
```typescript
// Example test structure
describe('ZodiacService', () => {
  it('calculates correct Chinese zodiac for 1990', () => {
    const result = calculateChineseZodiac(1990);
    expect(result.animal).toBe('Horse');
  });
  
  it('calculates correct Indian zodiac for March 15', () => {
    const date = new Date(2000, 2, 15);
    const result = calculateIndianZodiac(date);
    expect(result.sign).toBe('Meena');
  });
  
  it('handles date edge cases correctly', () => {
    const date = new Date(2000, 0, 20); // Birch/Rowan boundary
    const result = calculateCelticZodiac(date);
    expect(result.tree).toBe('Birch');
  });
});
```

### Integration Tests
- Birthday input → zodiac calculation → PDF generation
- Print job submission → CUPS queue verification
- Full user journey simulation

### Manual Testing Checklist
- [ ] Touch input responsiveness
- [ ] Date validation (invalid dates, leap years)
- [ ] PDF generation quality
- [ ] Print output clarity
- [ ] Loading animations smoothness
- [ ] Screen transitions
- [ ] Sound effects functionality
- [ ] Fold pattern accuracy

---

## 10. Documentation & Handoff

### User Manual
- Kiosk operation instructions
- Printer maintenance
- Troubleshooting common issues
- Folding instructions for zine

### Developer Documentation
- API endpoint specifications
- Database schema (if expanded)
- Configuration options
- Deployment procedures

### Academic Documentation
- Theoretical framework references
- Design decisions rationale
- User testing results
- Technical challenges & solutions

---

## Conclusion

This software design provides a complete blueprint for ZodiZine, covering:
- ✅ All three zodiac calculation algorithms
- ✅ PDF generation with proper zine layout
- ✅ Frontend-backend integration
- ✅ Hardware configuration
- ✅ Deployment procedures
- ✅ Future enhancement roadmap

The modular architecture allows for incremental development while maintaining code quality and system reliability. The design balances technical sophistication with the project's core principle: transforming simple birthday input into meaningful, collectible experiences.

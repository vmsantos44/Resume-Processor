# 📄 Resume Processor & Interpreter Candidate Management System

A comprehensive web application for processing interpreter/translator resumes with AI-powered parsing, tier-based scoring, workflow management, and CRM integration.

## 🚀 Features

### Core Functionality
- 📤 **File Upload**: Support for PDF, DOC, and DOCX formats with drag-and-drop
- 🔍 **Text Extraction**: Advanced extraction using PyPDF2, pdfplumber, python-docx, and DOCX→PDF conversion
- 🤖 **AI-Powered Parsing**: OpenAI GPT-4o-mini for structured data extraction
- 🚫 **Duplicate Detection**: Email-based identification with MD5 hash fallback
- 🌍 **Location Classification**: Automatic Onshore/Offshore detection based on address and phone

### Tier Scoring System
- **Tier 1**: Remote interpreting experience (VRI/OPI) + Score ≥80
- **Tier 2**: On-site only interpreting (capped at Tier 2)
- **Tier 3**: No interpreting experience (Score = 0)
- **Dynamic Scoring**: Configurable points for years of experience, certifications, QA/training, LSP experience

### Workflow Management
- ✅ **Status Tracking**: uploaded → processed/failed with retry logic (max 3 attempts)
- 🔄 **Automatic Retry**: Failed AI parsing retries automatically
- 📊 **Admin Dashboard**: Real-time monitoring with auto-refresh
- ⚙️ **Dynamic Settings**: Configure scoring rules without code changes
- 🔗 **CRM Integration**: Zoho Flow webhook for processed candidates

### Data Extraction
- Personal: Name, Email, Phone/Mobile, Address (with Onshore/Offshore classification)
- Languages: Primary language, other spoken languages
- Professional: Tier level, tier score, qualification status, role relevance
- Experience: Work history, certifications, skills
- Training: Training needed flag, processing notes

## 📋 Prerequisites

- Python 3.8+
- OpenAI API Key
- (Optional) Zoho Flow webhook URL for CRM sync

## 🛠️ Installation

### Local Setup

1. **Clone the repository**
```bash
git clone https://github.com/vmsantos44/Resume-Processor.git
cd Resume-Processor
```

2. **Install dependencies**
```bash
pip install -r requirements.txt
```

3. **Configure environment variables**
```bash
cp .env.example .env
```

Edit `.env` and add:
```
OPENAI_API_KEY=your_openai_api_key_here
ZOHO_FLOW_WEBHOOK=https://flow.zoho.com/your/webhook/url
```

4. **Run the application**
```bash
python app.py
```

5. **Access the application**
- Upload Interface: http://localhost:5001
- Admin Dashboard: http://localhost:5001/dashboard
- Settings Page: http://localhost:5001/settings/page

### Replit Deployment

1. **Import to Replit**
   - Go to [Replit](https://replit.com)
   - Click "Create Repl" → "Import from GitHub"
   - Paste: `https://github.com/vmsantos44/Resume-Processor.git`

2. **Configure Secrets**
   - In Replit, go to "Tools" → "Secrets"
   - Add:
     - `OPENAI_API_KEY` = your OpenAI API key
     - `ZOHO_FLOW_WEBHOOK` = your Zoho webhook URL (optional)

3. **Run**
   - Click the "Run" button
   - Replit will automatically install dependencies and start the server

## 📁 File Structure

```
Resume-Processor/
├── app.py                      # Flask backend with all logic
├── index.html                  # Upload interface
├── dashboard.html              # Admin dashboard
├── settings.html               # Scoring settings UI
├── requirements.txt            # Python dependencies
├── .env                        # Environment variables (not in git)
├── .env.example               # Template for .env
├── .gitignore                 # Git ignore rules
├── .replit                    # Replit configuration
├── scoring_settings.json      # Dynamic scoring configuration
├── candidates_db.json         # Candidate database (auto-generated)
└── resumes/                   # Uploaded resumes storage
    └── converted_pdfs/        # DOCX→PDF conversions
```

## 🎯 Usage

### 1. Upload a Resume
1. Navigate to http://localhost:5001
2. Drag & drop a resume or click to browse
3. Click "Process Resume"
4. View extracted data with tier classification

### 2. Monitor Candidates
1. Go to http://localhost:5001/dashboard
2. View all candidates with filtering options:
   - All
   - Processed
   - Needs Processing
   - Needs Attention (failed)
   - Duplicates
3. Retry failed candidates with one click

### 3. Configure Scoring
1. Go to http://localhost:5001/settings/page
2. Adjust scoring rules:
   - Points for 5+ years experience
   - Points for certifications
   - Points for QA/Training
   - Points for LSP experience
3. Update tier thresholds (Tier 1 min, Tier 2 min)
4. Manage known LSPs and remote keywords
5. Click "Save Settings"

## 🔌 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/` | Upload interface |
| `POST` | `/upload` | Upload and process resume |
| `GET` | `/dashboard` | Admin dashboard UI |
| `GET` | `/candidates` | Get all candidates (JSON) |
| `GET` | `/candidates?status=processed` | Filter by status |
| `POST` | `/retry/{candidate_id}` | Retry failed candidate |
| `GET` | `/settings` | Get scoring settings (JSON) |
| `POST` | `/settings` | Update scoring settings |
| `GET` | `/settings/page` | Settings UI |

## ⚙️ Configuration

### Scoring Settings (`scoring_settings.json`)

```json
{
  "version": "1.0",
  "scoring_rules": {
    "years_5plus": 30,
    "certifications": 20,
    "qa_training": 10,
    "lsp_experience": 10
  },
  "tier_thresholds": {
    "tier_1_min": 80,
    "tier_2_min": 60
  },
  "known_lsps": ["LanguageLine", "TransPerfect", "Propio", "Lionbridge"],
  "remote_keywords": ["VRI", "OPI", "Remote Interpreting", "Phone Interpreting"]
}
```

### Tier Assignment Logic

1. **Tier 1** (Remote-Ready)
   - Has remote interpreting experience (VRI/OPI/Phone/Video)
   - Score ≥80 (configurable)
   - Training needed: false

2. **Tier 2** (On-site Only)
   - Has on-site interpreting experience (court, hospital, community)
   - No remote experience
   - Capped at Tier 2 even if score ≥80
   - Training needed: true

3. **Tier 3** (No Experience)
   - No interpreting experience (translators, HR, IT, etc.)
   - Score = 0
   - Training needed: true

### Onshore/Offshore Classification

Automatically determined by:
- ✅ U.S. state name in address → Onshore
- ✅ U.S. ZIP code (5-digit or 5+4) → Onshore
- ✅ U.S. phone format (+1 or 10-digit) → Onshore
- ❌ None of the above → Offshore

## 🔄 Workflow

1. **Upload** → Resume uploaded, text extracted, email identified
2. **Parse** → OpenAI processes resume into structured JSON
3. **Validate** → Check required fields (name, email, tier_level, tier_score, qualify)
4. **Classify** → Determine Onshore/Offshore based on address/phone
5. **Store** → Save to candidates_db.json with status
6. **Sync** → Send to Zoho Flow webhook (if configured and processed successfully)
7. **Retry** → Auto-retry if failed (max 3 attempts), then mark as 'failed'

## 🚨 Error Handling

- **Missing Fields**: Retry up to 3 times, then mark as failed
- **Invalid JSON**: Retry with error context
- **Duplicate Email**: Skip processing, return existing data
- **Text Extraction Failure**: Return error message
- **DOCX Conversion**: Fallback to direct docx extraction if PDF conversion fails

## 🔒 Security Notes

- Never commit `.env` file to Git (already in `.gitignore`)
- Store API keys in environment variables or Replit Secrets
- Run in production with a WSGI server (Gunicorn, uWSGI) instead of Flask debug mode
- Consider adding authentication for admin endpoints in production

## 📊 Data Schema

### Candidate Record
```json
{
  "id": "email@example.com",
  "filename": "Resume.pdf",
  "status": "processed",
  "retry_count": 0,
  "synced": true,
  "scoring_version": "1.0",
  "uploaded_at": "2025-09-24T12:00:00",
  "processed_at": "2025-09-24T12:00:30",
  "zoho_synced_at": "2025-09-24T12:00:31",
  "parsed_data": { /* see below */ }
}
```

### Parsed Data (AI Output)
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "primary_language": "Spanish",
  "other_spoken_languages": ["English", "Portuguese"],
  "service_location": "Onshore",
  "mobile": "+1 510 239 6650",
  "remote_experience": true,
  "tier_level": "Tier 1",
  "tier_score": 90,
  "education": "Bachelor's Degree",
  "qualify": "Yes - Qualified",
  "role_relevance": "Interpreter",
  "training_needed": false,
  "processing_notes": "Experienced remote interpreter with VRI/OPI",
  "certifications": ["CHI - Court Interpreter"],
  "skills": ["Medical Interpreting", "Legal Interpreting"],
  "experience": [
    {
      "company": "LanguageLine Solutions",
      "position": "Remote Interpreter",
      "duration": "2018 - Present",
      "description": "VRI and OPI services"
    }
  ],
  "address": {
    "street": "123 Main St",
    "city": "Oakland",
    "state": "CA",
    "zip_code": "94601",
    "country": "USA"
  }
}
```

## 🐛 Troubleshooting

### Port 5000 Already in Use
- macOS AirPlay Receiver uses port 5000 by default
- This app uses port 5001 instead

### DOCX Email Extraction Issues
- App converts DOCX→PDF first for better text extraction
- Applies regex cleanup for broken emails (e.g., "name @ gmail . com" → "name@gmail.com")

### OpenAI API Errors
- Check API key is valid in `.env`
- Ensure you have credits in your OpenAI account
- Check rate limits if processing many resumes

### Missing tier_score = 0
- Fixed: Validation now allows 0 as valid tier_score (for Tier 3 candidates)

## 📝 License

MIT License - feel free to use and modify for your needs.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 📧 Support

For issues or questions, please open an issue on GitHub.

---

Built with Flask, OpenAI GPT-4o-mini, and modern web technologies.

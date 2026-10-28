# NutriNex: Google AI Studio Build Prompt

Paste everything inside the box below into AI Studio > Build. Then use the follow-up prompts at the end, one at a time.

---

```
Build "NutriNex", a mobile-first progressive web app (PWA) that is a closed-loop digital health companion for the Malaysian public, including people with or at risk of diabetes. It bridges clinical data (health screening reports) and daily eating behaviour using Gemini. Tagline: "Snap. Predict. Act." Positioning: zero-friction, localized (Malaysian food), low cost, non-restrictive (balance, not guilt).

## 1. USERS AND PURPOSE
Adults in Malaysia, many with low nutritional literacy, some managing diabetes/pre-diabetes, plus their family caregivers. UI languages: English and Bahasa Malaysia (toggle in settings; AI replies follow the chosen language). Use simple, warm, jargon-free wording.

## 2. CORE USER FLOWS (bottom tab bar: Home, Snap, Reports, Care, Profile)

A) PROGRESSIVE PROFILE SETUP
- Onboarding asks only 4 things first: age, sex, height, weight. The app is usable immediately.
- Later, contextual cards ask for activity level, goal (lose / maintain / gain), conditions (diabetes, pre-diabetes, hypertension, none), medications, allergies, halal/vegetarian preference. Show a "profile completeness" ring.
- TDEE calculator: Mifflin-St Jeor BMR (male: 10W+6.25H-5A+5; female: 10W+6.25H-5A-161), times activity factor (1.2, 1.375, 1.55, 1.725, 1.9). Goal adjustment: lose -15%, gain +10% (never below 1,200 kcal women / 1,500 kcal men). Show BMR, TDEE, target calories, and macro targets (protein g/kg by goal, fat 25-30%, carbs remainder), plus fibre target (14 g per 1,000 kcal). Show the formula steps in a "How is this calculated?" drawer. Recalculate whenever the profile changes.

B) SNAP & TRACK (food scan)
- Inputs: take a photo with the camera, upload from gallery, or scan a barcode (use the BarcodeDetector API with a manual-entry fallback and look up Open Food Facts).
- Voice logging: tap the mic and say "nasi lemak with two eggs and teh tarik" (Web Speech API; Gemini parses the text). Support English and Malay.
- Send the image to a Gemini multimodal model. The AI must (1) segment the plate into separate components, (2) identify each dish, with strong coverage of Malaysian foods (nasi lemak, roti canai, char kuey teow, laksa, mee goreng, nasi kandar, satay, teh tarik, kuih, etc.), (3) estimate portion size in grams, (4) estimate per component and in total: calories, protein, carbs, fat, fibre, sugar, sodium, glycaemic index and glycaemic load, (5) give a confidence value per item.
- Return STRICT JSON using a response schema:
  { "items":[{"name","name_ms","portion_g","kcal","protein_g","carbs_g","fat_g","fibre_g","sugar_g","sodium_mg","gi","gl","confidence"}], "totals":{...}, "meal_gl_band":"low|medium|high", "nudge":"string", "swap_tip":"string" }
- Show the photo with a result card: macro ring, per-item list, and a glycaemic gauge (Low/Medium/High). User can edit any item or portion (slider) before saving to the day log. Always label values "AI estimate".
- "Glycemic Flush Nudge": if meal GL is medium/high, show a specific, friendly action (e.g. "A 10-15 minute walk now can help flatten your blood-sugar spike" or a smarter swap such as less rice / add vegetables / skip sweetened drink). Offer a one-tap "Remind me to walk in 15 min" that creates a reminder.

C) HEALTH SCREENING REPORT ANALYZER
- Upload a PDF or photo of a lab report (or take a photo). Gemini extracts every test: name, value, unit, reference range, flag.
- Return STRICT JSON: { "tests":[{"name","value","unit","ref_range","status":"normal|borderline|high|low|critical","explanation_simple"}], "summary", "top_3_findings", "diet_advice":[], "lifestyle_advice":[], "questions_for_doctor":[], "urgent":boolean }
- Cover common panels: fasting glucose, HbA1c, lipid profile, kidney (creatinine, eGFR, urea), liver enzymes, uric acid, full blood count, blood pressure, BMI.
- Explain results in plain language in the user's language, with colour-coded status chips and a trend chart when the user has 2+ reports of the same test.
- If any value is critical or "urgent" is true, show a prominent red banner: "Please see a doctor promptly" and offer to alert family contacts (see E).
- Never diagnose, never change medication doses. End every analysis with "This is educational information, not a medical diagnosis. Discuss results with your doctor."
- Add "Export clinician summary" (printable one-page PDF with report findings, 30-day nutrition and score trends).
- Privacy: tell the user the file is processed for analysis; do not keep the original file unless they tick "Save to my vault". Offer one-tap delete.

D) DAILY BALANCE SCORE, MILESTONES AND REWARDS
- Home shows today's progress rings for calories, protein, carbs, fat, fibre, and water.
- Daily Balance Score (0-100), calculated transparently: 30% calories within +/-10% of target; 25% macro balance (protein, fibre met; fat and carbs within range); 20% glycaemic quality (share of meals with low/medium GL); 15% logging completeness (meals logged vs expected); 10% activity/hydration. Show a breakdown bar so users see exactly what moved the score. The tone must reward balance, never shame ("toxic restriction" is the opposite of our goal).
- Streaks (days with score >= 70), weekly average, and a score history chart.
- Milestones with badges and unlockable rewards: 50 / 100 / 250 / 500 / 1,000 cumulative points; 3 / 7 / 14 / 30-day streaks; first healthy-swap; first report analysed. Levels: Seed, Sprout, Bloom, Thrive. Rewards: celebratory animation, new badge, unlockable themes/avatars, and a shareable achievement card (image). Do not promise real-world prizes; keep a "partner rewards (coming soon)" placeholder.

E) REMINDERS, ALARMS AND FAMILY EMERGENCY
- Reminders: medication (name, dose text, times, repeat days), meals, water, post-meal walk, doctor appointments (date, place, notes). Calendar-style agenda view. Also let the user say a reminder by voice ("remind me to take metformin at 8 pm every day") and have Gemini convert it to a structured reminder.
- Alarms: full-screen ringing alert with sound and vibration, Snooze / Taken / Skip buttons. Log adherence (taken/missed) and include it in the clinician summary. Use the Notifications API + service worker + Web Audio. Because browsers cannot guarantee background alarms, show a clear one-time notice and an "Install to home screen" prompt for best reliability.
- Care Circle: user adds up to 5 family members/caregivers (name, relationship, phone, WhatsApp). Each gets a consent step.
- SOS button (always reachable from every screen, press-and-hold 2 seconds with a 5-second cancel countdown): captures GPS via Geolocation, builds a Google Maps link, and opens WhatsApp / SMS / phone call to the selected contacts with a pre-filled message: name, time, location link, key medical info (conditions, medications, allergies, blood type if provided). Also offer "Call 999". Add a "Check-in" timer: if the user does not confirm "I'm OK" within a set time, prompt to alert family.
- Auto-alert option: when the report analyzer flags critical values or the user misses 3 consecutive medication alarms, suggest alerting a caregiver (always ask; never send silently).
- Optional caregiver view: a read-only shareable link/QR showing today's score, medication adherence, and last SOS (mock with local data if no backend).

F) LIVE DATA AND AI COACH
- Use Gemini with Google Search grounding for up-to-date nutrition facts of Malaysian foods/packaged products and for current Malaysian dietary guideline references (e.g. Malaysian Dietary Guidelines, MOH diabetes guidance). Show source links for grounded answers.
- "Ask NutriNex" chat: voice or text, context-aware (profile, today's log, latest report). Suggests Malaysian-friendly swaps and next-meal ideas that fit the remaining targets.
- Weekly insight card: a 3-sentence AI summary of patterns (e.g. "Your high-GL meals mostly happen at dinner").

## 3. DATA AND TECH REQUIREMENTS
- React + TypeScript + Tailwind. Gemini API via the @google/genai SDK; use a vision-capable Gemini model for food/report analysis, structured output (responseMimeType application/json + responseSchema) and a faster model for chat. Put model names in one config file so they are easy to change.
- All AI calls go through one service module with: loading states, retries, timeouts, JSON validation, and friendly error messages. Never crash on bad AI output.
- Persistence: encrypt-at-rest style local storage using IndexedDB (with Web Crypto AES-GCM for health data). Provide Export my data (JSON) and Delete all my data. Structure the data layer behind an interface so it can be swapped for Firebase/Firestore later.
- Public-use compliance: Malaysian PDPA-aware design. First-launch consent screen (what is collected, why, how to delete), separate explicit consent for health data and location. Collect the minimum. No ads, no selling data. Privacy page in settings.
- Accessibility: large tap targets, high contrast, font-size setting, screen-reader labels, works one-handed on a 360px-wide phone, and works offline for viewing logs (AI features show "needs internet").
- Add a demo mode with sample data (a sample nasi lemak photo result, sample lab report) so every screen can be tested without real data.

## 4. VISUAL STYLE
Calm, trustworthy clinical-meets-friendly. Deep navy (#0F2A4A) for headers and text, teal (#2A9D8F) as the primary accent, warm orange (#F77F00) for calls to action and nudges, off-white background (#F5F5F0). Rounded cards, soft shadows, camera-viewfinder corner brackets on the Snap screen, a semicircular glycaemic gauge (green/amber/red), smooth progress-ring animations. Font: Inter. Dark mode supported. Subtle haptic/animation feedback on rewards. The SOS button is a distinct red, never confused with other buttons.

## 5. WHAT IT MUST NOT DO
- Do not diagnose disease, prescribe, or advise changing or stopping medication; always route to a doctor for that.
- Do not shame, set extreme restriction targets, or show body-weight-loss pressure; no calorie targets below the safety floors stated above.
- Do not state AI estimates as exact; always show confidence and allow edits.
- Do not store or transmit health data, photos, or location without consent; never send an SOS silently.
- Do not hardcode or expose the API key in client code or logs.
- Do not use placeholder lorem ipsum; use realistic Malaysian examples (RM, local foods, local clinic names as sample text).

## BUILD ORDER
Scaffold the full navigation and design system first, then implement in this order: Profile + TDEE, Snap & Track, Daily Balance Score, Report Analyzer, Reminders/Alarms, Care Circle + SOS, AI Coach. Make every screen functional with demo data before wiring Gemini.
```

---

## Follow-up prompts (send one at a time after the first build runs)

1. "Test the Snap & Track flow with a sample nasi lemak photo. Fix any JSON parsing errors and add a manual edit sheet for each item."
2. "Improve the Daily Balance Score breakdown screen and add the milestone unlock animation and shareable achievement card."
3. "Harden the Report Analyzer: handle multi-page PDFs, blurry photos, and values in different units (mmol/L vs mg/dL). Add the critical-value banner logic."
4. "Make reminders reliable: add a service worker, PWA manifest, install prompt, and the full-screen alarm with snooze/taken/skip and adherence log."
5. "Complete the SOS flow: hold-to-trigger, cancel countdown, GPS link, WhatsApp/SMS/call buttons, and the check-in timer."
6. "Add Bahasa Malaysia across the whole UI and AI responses."
7. "Move Gemini calls behind a backend proxy (Cloud Run) before public launch, then deploy."

## Practical notes

- **Public launch:** AI Studio's in-editor preview is for building. For real public use, deploy (Cloud Run from AI Studio, or export to GitHub) and put the Gemini key behind a backend so it cannot be extracted from the browser. This is the most important step before launch.
- **Web limits:** A web app cannot silently send SMS, guarantee background alarms, or read Apple Watch / CGM data. The prompt works around this with install-to-home-screen, user-confirmed WhatsApp/SMS, and clear notices. True always-on alarms and wearable sync need a native wrapper later.
- **Health content:** Have a Malaysian dietitian or doctor review the nutrition database, score rules, and critical-value thresholds, and get legal advice on PDPA and medical-device classification before charging users or making health claims.

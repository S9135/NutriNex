# NutriNex Implementation Plan

NutriNex is a mobile-first PWA digital health companion for the Malaysian public.

## 1. Project Scaffolding
- [x] Configure React + Vite with Tailwind CSS.
- [x] Add PWA configuration (`vite-plugin-pwa`).
- [ ] Implement foundational navigation structure.

## 2. Implementation Roadmap (Order of Operations)
1. **Scaffold & Navigation**: Set up bottom tab bar (Home, Snap, Reports, Care, Profile) using `react-router-dom`.
2. **Profile Setup**: Build onboarding and TDEE calculator component.
3. **Snap & Track**: Integrate camera/upload and Gemini vision API for food analysis.
4. **Daily Balance Score**: Implement calculation logic and progress visualization.
5. **Report Analyzer**: Build PDF/image uploader and Gemini analysis logic.
6. **Reminders & Alarms**: Implement Notifications API + PWA worker.
7. **Care Circle & SOS**: Build emergency flow and caregiver alert features.
8. **AI Coach**: Integrate Gemini chat with search grounding.

## 3. Data & Tech
- **AI**: Google GenAI SDK (server-side proxy).
- **Persistence**: IndexedDB (client-side) for health data.
- **Languages**: Toggle between English and Bahasa Malaysia.

---
Would you like to proceed with the Navigation scaffold?

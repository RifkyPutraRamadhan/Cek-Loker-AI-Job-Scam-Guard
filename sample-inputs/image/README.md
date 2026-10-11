# Image Input Samples: Cek Loker Multimodal Testing

Use the image files provided in this folder to test the vision (multimodal) capabilities of the Cek Loker agent. Simply upload or drag-and-drop the image into the chat interface to begin the assessment.

The content inside these posters is identical to the text samples, designed to prove that the AI evaluates text and images with the exact same accuracy.

## 1. Low Risk Scenario (Target Score: 0 | Range: < 20)
**File:** `poster-cek-loker-1-low.jpg`

**Evidence Context:** The AI should successfully read the company name, position, and the official email domain from the poster. It will verify the company as registered and detect the "free of charge" statement.

---

## 2. Medium Risk Scenario (Target Score: 45 | Range: 20 - 49)
**File:** `poster-cek-loker-2-medium.jpg`

**Evidence Context:** The AI should extract the free email address (gmail) and the unregistered company name from the image, scoring it correctly without hallucinating data.

---

## 3. High Risk Scenario (Target Score: 75 | Range: 50+)
**File:** `poster-cek-loker-3-high`

**Evidence Context:** The AI must read the deposit fee requirement, the unregistered entity, and the free email (yahoo) from the poster. This will test if the image successfully triggers the automatic document and email draft generation.
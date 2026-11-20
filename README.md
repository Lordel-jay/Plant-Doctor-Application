# Plant Doctor — AI Plant Disease Detector

Kotlin + CameraX + TensorFlow Lite. Open this folder as a project in Android Studio (Hedgehog+).

## What's already built
- CameraX preview + capture (`MainActivity`)
- Generic TFLite image-classifier wrapper (`ml/PlantClassifier.kt`) — works with any
  compatible `.tflite` model, no code changes needed to swap models
- Treatment lookup from a local JSON file (`assets/treatments.json`), with 8 sample
  entries (tomato/potato/apple) to prove the flow end-to-end
- Scan history via Room (`db/`), shown in `HistoryActivity`
- Result screen showing diagnosis + symptoms + treatment + prevention (`ResultActivity`)

## What YOU need to add before it runs: the model
The app expects two files in `app/src/main/assets/`:
- `model.tflite` — an image classification model
- (labels are read from the model's embedded metadata if present; the bundled
  `labels.txt` is there as a reference/fallback)

You have three options, fastest first:

### Option A — Teachable Machine (fastest, ~30 min, best for a working demo today)
1. Go to https://teachablemachine.withgoogle.com/ → "Image Project" → "Standard image model"
2. Create one class per disease you want to demo (e.g. "Tomato_Healthy", "Tomato_Early_Blight",
   "Tomato_Late_Blight") and upload/webcam-capture ~30-50 sample leaf photos per class
   (the PlantVillage dataset on Kaggle is a free source of sample leaf photos:
   https://www.kaggle.com/datasets/abdallahalidev/plantvillage-dataset)
3. Train in-browser, then Export Model → TensorFlow Lite → download
4. Drop the exported `model.tflite` into `app/src/main/assets/`
5. Update `treatments.json`'s `labelKey` fields to match your exact class names

This gets you a real on-device ML pipeline working today. Accuracy won't be
production-grade, but it proves every piece of the architecture.

### Option B — A pre-trained PlantVillage model (more classes, more realistic)
Several public repos have PlantVillage-trained models (38 disease classes across
14 crop species) you can convert to `.tflite` with `tf.lite.TFLiteConverter`:
- https://github.com/MarkoArsenovic/DeepLearning_PlantDiseases (the original PlantVillage/InceptionV3 training work)
- https://www.kaggle.com/datasets/abdallahalidev/plantvillage-dataset (dataset + several public notebooks)

If you go this route, expand `treatments.json` to cover the full label list the
model outputs (see `assets/labels.txt` for a starter subset covering tomato,
potato, and apple — extend it to match whichever classes your model supports).

### Option C — Google's CropNet (single crop, very high quality, if narrowing scope is OK)
https://tfhub.dev/google/cropnet/classifier/cassava_disease_V1/2 — an
official Google model, cassava-only, 88% accuracy, has a documented TFLite
conversion path. Good if you're open to narrowing the pitch to one crop.

## Authentication (Firebase) — setup required before this compiles
Login/register screens are built (`LoginActivity`, `RegisterActivity`), gating access to
`MainActivity`. Firebase project creation has to happen in your own Google account —
here's the one-time setup:

1. Go to https://console.firebase.google.com/ → "Add project" (free Spark plan is fine)
2. In the project, click "Add app" → Android → use package name `com.example.plantdoctor`
   (must match `namespace`/`applicationId` in `app/build.gradle` exactly)
3. Download the generated **`google-services.json`** and place it at `app/google-services.json`
   (same folder as `app/build.gradle` — the project won't build without this file)
4. In the Firebase console, go to Build → Authentication → "Get started" →
   enable the **Email/Password** sign-in provider
5. Build and run — you should be able to sign up a test account and log in

Once this is in place, `LoginActivity` is the launcher screen; a signed-in session
persists across app restarts (Firebase keeps you logged in until you tap the
logout icon in the top-right of the camera screen).

## Next things to wire up together
1. Pick a model option above and get `model.tflite` into `assets/`
2. Run on a real device (camera doesn't work in most emulators)
3. Expand `treatments.json` to cover your model's full class list
4. Polish: loading spinner during inference, confidence threshold tuning,
   maybe a "not sure this is a leaf" guard before running the model


## Professional UI refresh

The Android layouts have been refreshed with a consistent botanical visual system:
- Forest-green and emerald palette with a soft off-white app background
- Updated sign-in and registration screens with rounded input fields and clear hierarchy
- Camera scanner overlay with a focused scan action and quick history/logout controls
- Diagnosis result cards for the captured leaf, confidence, symptoms, treatment and prevention
- A cleaner scan-history list with more readable scan details
- Updated application theme and a vector Plant Doctor launcher icon

The existing Firebase, CameraX, TensorFlow Lite, Room and treatment lookup logic has been retained. The XML view IDs used by View Binding have also been preserved.

## Before building

This ZIP does not contain `app/google-services.json` or `app/src/main/assets/model.tflite`. Add the Firebase configuration file for your own Firebase Android app and the compatible TensorFlow Lite model before running the full application. The model's output labels must match the keys in `assets/treatments.json` / `assets/labels.txt`.

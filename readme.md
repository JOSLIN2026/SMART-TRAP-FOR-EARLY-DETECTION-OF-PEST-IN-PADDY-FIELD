

## 1. The physical trap

Your trap needs to do two things: attract pests and photograph them.

For brown planthopper (BPH), stem borer moths, and leaf folder moths, a UV LED at 365–395nm is the right wavelength — these pests are strongly phototactic. Mount a sticky trap board or a collection funnel below the UV lamp. The ESP32-CAM points directly at the collection area and triggers a photo every few minutes or when motion is detected via a PIR sensor.

Your existing dual-ESP32 architecture (ESP-WROOM-32 + ESP32-CAM) is perfect for this: one chip handles sensing and comms (DHT22, SIM800L, WiFi), the other focuses entirely on camera and inference.

---

## 2. Building the AI model

This is the core. Here's the practical path:

**Step 1 — Collect or find a dataset**

You need labeled images of your target pests. Look for:
- [IP102 dataset](https://github.com/xpwu95/IP102) — 75,000+ insect images across 102 categories including BPH, stem borer, leaf folder
- [Rice pests Roboflow datasets](https://roboflow.com) — search "rice pest", several are public
- Your own trap photos (most valuable — matches your exact lighting conditions)

You need at least 200–300 images per pest class to start. More is better.

**Step 2 — Choose your model architecture**

| Model | Best for | Tradeoff |
|---|---|---|
| YOLOv5n / YOLOv8n | Real-time detection, bounding boxes | Needs more RAM (~4MB+) |
| MobileNetV2 | Classification (what pest is this?) | Smaller, faster on ESP32 |
| EfficientDet-Lite | Good accuracy, small size | Best balance for edge |

For your use case — classify which pest species and count them — **MobileNetV2 or EfficientDet-Lite** trained with TensorFlow/Keras is the practical first choice. YOLOv5 is the upgrade path when you need to detect multiple pests in one frame.

**Step 3 — Train the model**

Use Google Colab (free GPU). Workflow:
1. Upload dataset to Google Drive
2. Use Teachable Machine (quickest — drag images into classes, export TFLite directly) for a prototype
3. For production: train with TensorFlow + `tf.keras`, then export to TFLite with INT8 quantization

Teachable Machine is genuinely viable for a first working version — you can have a running model in one afternoon. YOLO is the upgrade path for higher accuracy.

**Step 4 — Deploy to ESP32**

The ESP32 can run TFLite Micro. Use the [ESP-NN optimized library](https://github.com/espressif/esp-nn) or the `esp32-camera` + `tflite-micro` Arduino library stack. The model needs to be:
- INT8 quantized (not float32)
- Under ~1.5MB for reliable operation
- Input image resized to 96×96 or 128×128

---

## 3. What the system does in the field

1. UV LED attracts pests at night
2. ESP32-CAM captures image every 10 minutes (configurable)
3. TFLite model runs inference on-device → outputs: pest species + count
4. Count + timestamp + temp/humidity sent to Firebase
5. If count crosses threshold (e.g. >5 BPH per image), SIM800L sends SMS to farmer
6. SMS includes pest name, count, and a natural remedy tip in Malayalam (you can hardcode remedy strings per pest species)

---

## 4. The pests and their detection signatures

| Pest | Detection cue | UV attraction |
|---|---|---|
| Brown planthopper (BPH) | Small, oval, brown/black body | Very high |
| Stem borer moth | White/pale moth with dark spots | High |
| Leaf folder moth | Yellowish moth, ~10mm wingspan | High |

All three are strongly attracted to UV light, so a single trap catches all three. The model distinguishes them by shape, color, and size in the image.

---

## 5. Recommended toolchain

| Stage | Tool |
|---|---|
| Dataset labeling | Roboflow (free tier) |
| Training | Google Colab + TensorFlow |
| Quick prototype | Google Teachable Machine |
| Model conversion | TFLite converter (Python) |
| Firmware | Arduino IDE + ESP-IDF |
| Cloud | Firebase Realtime DB (free tier) |
| Dashboard | Blynk or a simple HTML page hosted on GitHub Pages |

---

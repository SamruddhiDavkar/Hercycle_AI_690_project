# 🌸 HerCycle AI
### PCOD Wellness Assistant with Food Image Recognition — Running on NVIDIA Jetson Nano
**DATA 690 — Real-Time AI Systems | Spring 2026 | UMBC Data Science M.S.**
**Author:** Samruddhi Davkar

---

## 📌 Project Status — Final Prototype

| Phase | Where it ran | Status | Details |
|---|---|---|---|
| Phase 1 — Preliminary results | Windows laptop (CPU) | ✅ Complete | [README_project_progress.md](README_project_progress.md) |
| **Final — On-device prototype** | **NVIDIA Jetson Nano (GPU)** | ✅ **Presented** | This README + [final presentation](docs/HerCycleAI_Presentation.pdf) |
| Phase 2 — Fine-tuning & optimization | Jetson Nano | 🔜 Planned | See [Next Steps](#-next-steps--phase-2) |

HerCycle AI is now a **working end-to-end prototype**: both models are loaded and running **at the same time on the Jetson Nano GPU**, fully on-device with **no internet required**.

📊 **Final presentation:** [PDF](docs/HerCycleAI_Presentation.pdf) · [PowerPoint](docs/HerCycleAI_Presentation.pptx)

---

## 🩺 The Problem

Polycystic Ovary Disorder (PCOD) is one of the most common hormonal disorders affecting women of reproductive age.

- **~1 in 5** women are affected globally
- **~70%** of cases are linked to insulin resistance
- There is no permanent cure. **Diet and lifestyle are the first line of management**: low-GI foods reduce insulin resistance and anti-inflammatory foods help lower androgen levels

**The gap today:** there's no personalized PCOD food tool, generic chatbots give vague answers, safe foods are hard to identify quickly, and dietitian visits are expensive. As a result, many women rely on guesswork every day.

## 💡 The Solution

| Component | What it does |
|---|---|
| 💬 **PCOD Chatbot** | Answers PCOD diet and lifestyle questions with grounded, PCOD-specific answers rather than generic ones |
| 📷 **Food Image Classifier** | Looks at a food image (upload or webcam) and labels it **PCOD-Friendly** or **PCOD-Unfriendly** with a confidence score and an explanation |
| ⚡ **Real-time, on-device** | Runs fully on the NVIDIA Jetson Nano: fast inference, no internet, complete privacy |
| 🕐 **Always available** | A personal PCOD wellness companion with no appointments and no waiting |

---

## 🧠 Models Used

![Models Used](docs/slides/slide-4.png)

| | Part A — Chatbot | Part B — Food Classifier |
|---|---|---|
| **Model** | Llama 3.2 3B | GoogLeNet (Inception v1) |
| **Runtime / Framework** | Ollama v0.22.0 | PyTorch + torchvision |
| **Quantization / Pretraining** | Q4_K_M (4-bit) | ImageNet (1.2M images) |
| **Size / Parameters** | 2.0 GB on device | 6,624,904 parameters |
| **Input** | Text question | 224 × 224 RGB image |
| **Output** | Text answer (~6–10 tok/sec on Jetson) | PCOD-Friendly / PCOD-Unfriendly |
| **Fine-tuned?** | No. Uses a PCOD system prompt + RAG | No. Uses ImageNet → PCOD keyword mapping (Food-101 fine-tuning planned) |

---

## 🟢 Final Results on Jetson Nano

### Running Model + Weights Loaded

![Running Model — Weights Loaded](docs/slides/slide-5.png)

| GoogLeNet (Food Classifier) | Ollama (Llama 3.2 3B Chatbot) |
|---|---|
| Weights: ImageNet pretrained ✓ | Ollama version 0.22.0 ✓ |
| Parameters: 6,624,904 | Model: Llama 3.2 3B, Q4_K_M |
| Load time: **3.42 s** | Size: 2.0 GB |
| Input: 224 × 224 × 3 (RGB) | Processor: **100% GPU** (confirmed with `ollama ps`) |
| Output: 2 PCOD classes | Context: 4096 tokens |
| Device: **CUDA (Jetson GPU)** | Server: port 11434 active |

✅ **Both models run simultaneously on the Jetson Nano GPU.**

### Live Demo

The full app was demonstrated live on the Jetson Nano: a **Gradio web UI** with two tabs (PCOD chatbot + food image classifier), served on port **7860** and reachable from any browser on the local network.

### Speed

| Component | Phase 1 (Windows CPU) | Final (Jetson Nano) | Target |
|---|---|---|---|
| Chatbot generation speed | 6.9 tok/sec avg | ~6–10 tok/sec | 8–10 tok/sec |
| Chatbot response time | 30.2 s avg | 30–40 s | < 15 s |
| Classifier model load | ~3–4 s | 3.42 s | — |
| Classifier inference | 244 ms / image | GPU (CUDA) | < 100 ms (TensorRT) |

### Classifier Metrics (10-image test set, pretrained baseline)

| Metric | Value | Target after fine-tuning |
|---|---|---|
| Accuracy | 70.0% | > 80% |
| Precision | 75.0% | > 75% ✓ already met |
| Recall | 60.0% | > 75% |
| F1 Score | 66.7% | > 75% |

Keyword mapping went from **50% → 60% → 70%** accuracy over two rounds of expansion. I stopped there on purpose: more tuning on 10 images would just overfit this test set. The full breakdown is in [README_project_progress.md](README_project_progress.md).

---

## 🔍 Key Findings

1. **An end-to-end, offline PCOD assistant is feasible on edge hardware.** A 4-bit 3B-parameter LLM and a CNN classifier fit and run together on the Jetson Nano GPU.
2. **Quantization makes the LLM practical on-device.** Q4_K_M keeps Llama 3.2 3B at 2.0 GB and runs it 100% on GPU.
3. **Pretrained ImageNet models can't tell foods apart well enough.** Keyword mapping tops out around 70%. The classifier gets the PCOD verdict right more often than the food name (see limitation #2), so fine-tuning on food-specific data is the clear next step.
4. **Latency is the main UX bottleneck.** Chatbot responses take 30–40 s, so cutting response time is the top priority for the next version.

---

## ⚠️ Current Limitations

![Current Limitations](docs/slides/slide-7.png)

| # | Limitation | Status |
|---|---|---|
| 01 | **Inference speed.** Chatbot responses take 30–40 s on the Jetson Nano. The target is < 15 s, via model-size tuning and TensorRT optimization. | 🟠 In Progress |
| 02 | **Food detection accuracy.** The webcam classifier identifies foods as generic categories. For example, a packet of chips was correctly flagged **PCOD-Unfriendly** but labeled as a generic package instead of chips. | 🔴 Needs Work |
| 03 | **Limited food categories.** Only tested on 10 manually downloaded images. Needs fine-tuning on Food-101 (20+ categories) to handle real-world variety. | 🔵 Phase 2 |
| 04 | **Image resolution & camera quality.** Low-resolution webcam frames reduce confidence. Better preprocessing and higher-resolution input should help, especially for packaged and processed foods. | 🔵 Phase 2 |

---

## 🚀 How to Run (Jetson Nano)

**1. Start the chatbot model (on the Jetson host):**
```bash
ollama pull llama3.2:3b-instruct-q4_K_M
ollama ps        # confirm the model shows 100% GPU once it's loaded
```
Ollama runs as a systemd service on the host at port `11434`.

**2. Run the app container:**
The app uses the `dustynv/l4t-pytorch:r35.2.1` base image. Run the container with `--network host` so it can reach Ollama on the host:
```bash
docker run --runtime nvidia -it --rm --network host \
  -v $(pwd):/workspace -w /workspace \
  dustynv/l4t-pytorch:r35.2.1 \
  bash -c "pip install -r requirements.txt && python app.py"
```

**3. Open the UI** at `http://<jetson-ip>:7860` from any browser on the same network.

## 🔭 Next Steps — Phase 2

| Goal | Plan | Expected result |
|---|---|---|
| Better food recognition | Fine-tune GoogLeNet on **Food-101** (20+ categories) | Accuracy 70% → > 80%, Recall 60% → > 75% |
| Faster chatbot | Model-size tuning + TensorRT optimization | Response time 30–40 s → < 15 s |
| Faster classifier | TensorRT optimization on the Jetson GPU | < 100 ms per image |
| Webcam robustness | Better preprocessing + higher-resolution input | Higher confidence on packaged and processed foods |

---

## ⚠️ Disclaimer

HerCycle AI is a research prototype built for academic purposes (DATA 690). It is **not a medical tool**. Always consult a qualified healthcare provider for PCOD diagnosis and treatment decisions.

---

*HerCycle AI | DATA 690 | Spring 2026 | UMBC Data Science M.S. | Samruddhi Davkar*

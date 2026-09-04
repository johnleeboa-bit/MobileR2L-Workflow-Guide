# 🤖 Daloy ng Paggawa sa GitHub - MobileR2L
## Sunod-Sunod na Proseso para sa Teacher-Student Model Training

---

## 📋 Gabay sa Pagsusulong

Ang dokumentong ito ay naglalaman ng kompletong hakbang-hakbang na instruksyon para sa MobileR2L workflow. Sumunod sa bawat yugto nang maingat.

---

## **YUGTO 1: I-Clone ang MobileR2L Repository**

### Hakbang 1.1: Pumunta sa GitHub Repository
- **URL:** https://github.com/snap-research/MobileR2L
- Buksan ang link sa inyong web browser

### Hakbang 1.2: I-Clone ang Repository
```bash
# Magbukas ng terminal/command prompt
git clone https://github.com/snap-research/MobileR2L.git

# Pumunta sa directory
cd MobileR2L
```

### Hakbang 1.3: I-Check ang Structure
```bash
# Tingnan ang mga folder
ls -la

# Dapat makita ninyo ang:
# - model/
# - teacher/
# - student/
# - scripts/
# - README.md
```

**✅ Tapos na ang Yugto 1**

---

## **YUGTO 2: I-Setup ang "Teacher" Model (NGP_PL)**

### Hakbang 2.1: Pumunta sa Teacher Folder
```bash
cd model/teacher/ngp_pl
```

### Hakbang 2.2: I-Install ang Dependencies
```bash
# Basahin ang README.md para sa specific requirements
cat README.md

# I-install ang Python packages
pip install -r requirements.txt

# Kung may environment.yml, gamitin ito
conda env create -f environment.yml
```

### Hakbang 2.3: I-Prepare ang Lego Dataset
```bash
# Siguraduhin na ang inyong lego dataset ay naka-organize
# Ang typical structure ay:
# datasets/
#   └── lego/
#       ├── images/
#       ├── annotations/
#       └── metadata.json
```

### Hakbang 2.4: Sanayan ang Teacher Model
```bash
# Patakbuhin ang training script
python train.py --dataset datasets/lego --epochs 50 --gpu 4

# ⏱️ INAASAHANG ORAS: Hanggang 15 minuto (gamit ang 4 GPU)
# 💾 OUTPUT: Trained model weights sa checkpoint/ folder
```

**Monitoran ang Training:**
- Tingnan ang logs para sa progress
- I-check ang GPU usage
- Siguraduhin na walang errors

**✅ Tapos na ang Yugto 2**

---

## **YUGTO 3: Gumawa ng Pseudo-Data**

### Hakbang 3.1: I-Prepare ang Script
```bash
# Balik sa main directory
cd ../../..

# Hanapin ang pseudo-data generation script
ls scripts/generate_pseudo_data.py
```

### Hakbang 3.2: Patakbuhin ang Pseudo-Data Script
```bash
# Patakbuhin ang script gamit ang trained teacher model
python scripts/generate_pseudo_data.py \
  --teacher-model model/teacher/ngp_pl/checkpoint/best_model.pth \
  --output-dir data/pseudo_data \
  --num-images 5000

# ⏱️ INAASAHANG ORAS: 30 minuto - 1 oras
# 💾 OUTPUT: 5,000 pseudo images sa data/pseudo_data/
```

### Hakbang 3.3: I-Verify ang Generated Data
```bash
# Tingnan ang generated images
ls -lh data/pseudo_data/

# Dapat may ~5,000 files
find data/pseudo_data -type f | wc -l
```

**✅ Tapos na ang Yugto 3**

---

## **YUGTO 4: I-Train ang "Student" Model (MobileR2L)**

### Hakbang 4.1: I-Setup ang Student Model Environment
```bash
# Siguraduhin na nasa main directory
cd /path/to/MobileR2L

# I-install ang student model requirements
pip install -r requirements.txt
```

### Hakbang 4.2: I-Prepare ang Training Configuration
```bash
# Buksan ang training config file
# Baguhin ang mga paths ayon sa inyong setup:
# - data_path: data/pseudo_data/
# - output_dir: experiments/lego_student/
# - batch_size: depende sa inyong GPU memory
# - learning_rate: default ay 0.001
```

### Hakbang 4.3: Simulan ang Student Model Training
```bash
# Patakbuhin ang training script
python train_student.py \
  --config configs/student_config.yaml \
  --data-dir data/pseudo_data/ \
  --output-dir experiments/lego_student/ \
  --batch-size 32 \
  --epochs 100

# ⏱️ INAASAHANG ORAS: 1-2 ARAW (depende sa GPU capacity)
# 💾 OUTPUT: Trained student model sa experiments/lego_student/
```

### Hakbang 4.4: Subaybayan ang Training Progress
```bash
# I-monitor ang tensorboard logs
tensorboard --logdir experiments/lego_student/logs

# Bubukas sa: http://localhost:6006
```

**📊 Mga Dapat Bantayan:**
- Training loss ay dapat bumaba
- Validation accuracy ay dapat tumaas
- Walang overfitting (training loss << validation loss)
- GPU memory usage ay stable

**✅ Tapos na ang Yugto 4**

---

## **YUGTO 5: I-Export ang Model para sa Android (ONNX)**

### Hakbang 5.1: I-Check ang Automatic Export
```bash
# Pagkatapos ng training, automatically mag-export sa ONNX
# Tingnan ang experiment folder

ls experiments/lego_student/

# Dapat makita:
# - best_model.pth
# - best_model.onnx  ✅
# - model_config.json
# - training_logs/
```

### Hakbang 5.2: I-Verify ang ONNX Model
```bash
# I-check ang file size at properties
file experiments/lego_student/best_model.onnx

# Dapat 10-50 MB ang laki (depende sa model architecture)
```

### Hakbang 5.3: I-Test ang ONNX Model Locally (Optional)
```bash
# I-install ang onnx tools
pip install onnx onnxruntime

# Patakbuhin ang test script
python scripts/test_onnx_model.py \
  --model experiments/lego_student/best_model.onnx \
  --test-image test_images/lego_sample.jpg
```

**✅ Tapos na ang Yugto 5**

---

## **YUGTO 6: I-Convert ang ONNX Model sa TFLite para sa Android**

### Hakbang 6.1: I-Install ang Conversion Tools
```bash
# I-install ang TensorFlow at conversion tools
pip install tensorflow tf2onnx onnx tf-lite-converter
```

### Hakbang 6.2: I-Convert ONNX to TFLite
```bash
# Patakbuhin ang conversion script
python scripts/convert_onnx_to_tflite.py \
  --onnx-model experiments/lego_student/best_model.onnx \
  --output-dir experiments/lego_student/tflite/ \
  --quantize true

# 💾 OUTPUT: best_model.tflite
```

### Hakbang 6.3: I-Verify ang TFLite Model
```bash
# I-check ang TFLite file
ls -lh experiments/lego_student/tflite/

# Dapat mas maliit kaysa ONNX (usually 50-75% smaller)
```

### Hakbang 6.4: Gamitin sa Android App
```bash
# I-copy ang TFLite model sa Android project
cp experiments/lego_student/tflite/best_model.tflite \
   /path/to/android/app/src/main/assets/models/

# Gamitin sa TensorFlow Lite Interpreter sa Android code
```

**✅ Tapos na ang Yugto 6 - TAPOS NA ANG LAHAT!**

---

## 📊 Buod ng Timeline

| Yugto | Proseso | Inaasahang Oras |
|-------|---------|-----------------|
| 1 | I-Clone ang Repository | 5 minuto |
| 2 | I-Setup at Sanayan ang Teacher | 15 minuto |
| 3 | Gumawa ng Pseudo-Data | 1 oras |
| 4 | I-Train ang Student Model | 1-2 ARAW |
| 5 | I-Export sa ONNX | 15 minuto |
| 6 | I-Convert sa TFLite | 30 minuto |
| **TOTAL** | | **~2-3 ARAW** |

---

## 🆘 Troubleshooting Guide

### Problem: "Out of Memory" Error sa Training
```bash
# Solusyon 1: Bawasan ang batch size
--batch-size 16  # mula 32

# Solusyon 2: Gumamit ng gradient accumulation
--gradient-accumulation-steps 2

# Solusyon 3: Gumamit ng mixed precision training
--mixed-precision true
```

### Problem: Model Hindi Converging
```bash
# Tingnan ang learning rate
# Subukan: 0.0001, 0.0005, 0.001, 0.01

# I-reset at i-retrain mula sa checkpoint
python train_student.py --resume-from checkpoint/latest.pth
```

### Problem: ONNX Conversion Failed
```bash
# I-check ang ONNX model validity
python -c "import onnx; model = onnx.load('model.onnx'); onnx.checker.check_model(model)"

# Kung may error, i-export ulit ang model
```

---

## 📚 Mga Helpful Resources

- **MobileR2L GitHub:** https://github.com/snap-research/MobileR2L
- **NGP Documentation:** https://github.com/snap-research/NGP_PL
- **TensorFlow Lite Guide:** https://www.tensorflow.org/lite/guide
- **ONNX Documentation:** https://onnx.ai/

---

## ✅ Checklist - Bago ang Deployment

- [ ] Teacher model trained successfully (15 min)
- [ ] 5,000 pseudo images generated
- [ ] Student model trained at converged
- [ ] Training loss graphs show good progress
- [ ] ONNX model exported successfully
- [ ] TFLite model created at tested locally
- [ ] Model file copied sa Android assets
- [ ] Android app builds successfully
- [ ] Model inference works sa device

---

## 📝 Notes

- **GPU Memory:** Minimum 8GB para sa student training
- **Disk Space:** ~50GB needed para sa training data at checkpoints
- **Python Version:** 3.8 o mas bago
- **CUDA Version:** 11.0+ para sa optimal performance

---

**Good luck sa inyong MobileR2L training! 🚀**

Kung may mga tanong, tingnan ang README files sa bawat folder.

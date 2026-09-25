# DPU-Docker-Basic
Master of Engineering - Artificial Intelligence and Data Engineering

# 🐳 Docker Basic for AI Engineers

> คอร์สพื้นฐาน Docker สำหรับวิศวกร AI / ML — เรียนรู้ตั้งแต่แนวคิดจนถึงการ deploy ML service จริง

**กลุ่มเป้าหมาย:** Data Scientist, ML Engineer, AI Engineer ที่เขียน Python เป็นแต่ยังไม่เคยใช้ Docker
**เวลาเรียนโดยประมาณ:** 4–6 ชั่วโมง
**สิ่งที่ต้องเตรียม:** เครื่องที่ติดตั้ง [Docker Desktop](https://www.docker.com/products/docker-desktop/) หรือ Docker Engine, พื้นฐาน Command Line, พื้นฐาน Python

---

## 📑 สารบัญ

| # | หัวข้อ |
|---|--------|
| 1 | [ทำไม Docker จึงสำคัญกับงาน AI Engineering](#1-ทำไม-docker-จึงสำคัญกับงาน-ai-engineering) |
| 2 | [Virtual Machine คืออะไร](#2-virtual-machine-คืออะไร) |
| 3 | [Container คืออะไร](#3-container-คืออะไร) |
| 4 | [Container vs Virtual Machine](#4-container-vs-virtual-machine) |
| 5 | [Image, Container, Registry](#5-image-container-registry) |
| 6 | [Core Commands & Dockerfiles](#6-core-commands--dockerfiles) |
| 7 | [คำสั่งหลักของ Docker](#7-คำสั่งหลักของ-docker) |
| 8 | [การเขียน Dockerfile](#8-การเขียน-dockerfile) |
| 9 | [Volumes & Networking](#9-volumes--networking) |
| 10 | [Volumes และการเก็บข้อมูลถาวร](#10-volumes-และการเก็บข้อมูลถาวร) |
| 11 | [Docker Networking และ Port Mapping](#11-docker-networking-และ-port-mapping) |
| 12 | [Compose & AI-Specific Tips](#12-compose--ai-specific-tips) |
| 13 | [Docker Compose: แอปแบบหลายคอนเทนเนอร์](#13-docker-compose-แอปแบบหลายคอนเทนเนอร์) |
| 14 | [Docker สำหรับงาน AI Workloads](#14-docker-สำหรับงาน-ai-workloads) |
| 15 | [Cheat Sheet & Final Challenge](#15-cheat-sheet--final-challenge) |
| 16 | [Docker Cheat Sheet](#16-docker-cheat-sheet) |
| 17 | [Final Challenge: Ship a Mini ML Service](#17-final-challenge-ship-a-mini-ml-service) |

---

## 1. ทำไม Docker จึงสำคัญกับงาน AI Engineering

### 😱 ปัญหาคลาสสิกที่ทุกคนเคยเจอ

> *"มันรันได้บนเครื่องผมนะ"* — ประโยคที่ทำให้ทีมทะเลาะกันมานักต่อนัก

งาน AI/ML มีความเปราะบางเรื่องสภาพแวดล้อมมากกว่างานซอฟต์แวร์ทั่วไป เพราะ:

- **Dependency Hell** — TensorFlow 2.15 ต้องการ Python 3.9–3.11, PyTorch ต้องการ CUDA เวอร์ชันเฉพาะ, บาง library ชนกันเอง
- **CUDA / cuDNN / Driver** — เวอร์ชันไม่ตรงกันแค่นิดเดียว GPU ก็ไม่ทำงาน
- **ผลการทดลองไม่ซ้ำเดิม (Non-reproducible)** — เทรนโมเดลเมื่อ 3 เดือนก่อนได้ accuracy 92% วันนี้รันโค้ดเดิมได้ 87%
- **ส่งงานให้ทีม DevOps ไม่ได้** — notebook รันได้ แต่ deploy ขึ้น production ไม่ผ่าน
- **ทีมใหม่เข้ามา setup เครื่อง 2 วัน** — กว่าจะลง environment ครบ

### ✅ Docker แก้ปัญหาเหล่านี้อย่างไร

| ปัญหา | Docker ช่วยอย่างไร |
|-------|---------------------|
| Environment ไม่ตรงกัน | แพ็ก OS + libraries + โค้ด ไว้ใน image เดียว รันที่ไหนก็เหมือนกัน |
| Reproducibility | Dockerfile = สูตรที่เขียนเป็นโค้ด เก็บใน Git ได้ ย้อนกลับได้ |
| Onboarding ช้า | `docker compose up` คำสั่งเดียวจบ |
| Deploy ยาก | Image เดียวกันใช้ได้ทั้ง local, staging, production, Kubernetes |
| หลายโปรเจกต์ชนกัน | แต่ละโปรเจกต์อยู่คนละคอนเทนเนอร์ ไม่ยุ่งกัน |
| GPU ใช้งานยาก | NVIDIA Container Toolkit ทำให้เข้าถึง GPU ได้ในคอนเทนเนอร์ |

### 🎯 สถานการณ์จริงที่ AI Engineer ต้องใช้ Docker

1. **Serve โมเดล** — ห่อ FastAPI + โมเดล เป็น image แล้ว deploy ขึ้น cloud
2. **Training Job** — ส่ง container ไปรันบน GPU cluster
3. **Data Pipeline** — รัน Airflow, Spark, database พร้อมกัน
4. **Experiment Tracking** — MLflow, Weights & Biases แบบ self-hosted
5. **CI/CD** — ทดสอบโมเดลอัตโนมัติในสภาพแวดล้อมที่ควบคุมได้

---

## 2. Virtual Machine คืออะไร

### 🖥️ นิยาม

**Virtual Machine (VM)** คือการจำลองเครื่องคอมพิวเตอร์ทั้งเครื่องขึ้นมาด้วยซอฟต์แวร์ โดยมี **Guest OS** เป็นของตัวเอง มี virtual CPU, RAM, disk และ network แยกออกจากเครื่องจริง

### 🏗️ สถาปัตยกรรม

```
┌─────────────────────────────────────────────┐
│   App A     │    App B     │     App C      │
├─────────────┼──────────────┼────────────────┤
│  Bins/Libs  │  Bins/Libs   │   Bins/Libs    │
├─────────────┼──────────────┼────────────────┤
│  Guest OS   │   Guest OS   │   Guest OS     │  ← หนัก 1–20 GB ต่อตัว
├─────────────┴──────────────┴────────────────┤
│              Hypervisor                     │  ← VMware, VirtualBox, Hyper-V, KVM
├─────────────────────────────────────────────┤
│              Host OS (อาจไม่มีใน Type 1)     │
├─────────────────────────────────────────────┤
│         Physical Hardware (Server)          │
└─────────────────────────────────────────────┘
```

### 🔧 ประเภทของ Hypervisor

| ประเภท | ลักษณะ | ตัวอย่าง |
|--------|--------|----------|
| **Type 1 (Bare-metal)** | ติดตั้งบนฮาร์ดแวร์โดยตรง ประสิทธิภาพสูง | VMware ESXi, Xen, KVM, Hyper-V Server |
| **Type 2 (Hosted)** | ติดตั้งบน OS ปกติอีกที | VirtualBox, VMware Workstation, Parallels |

### 👍 ข้อดีของ VM

- **แยกขาดจากกันสมบูรณ์ (Strong Isolation)** — VM พังไม่กระทบตัวอื่น ปลอดภัยสูง
- **รัน OS ต่างชนิดได้** — รัน Windows บน Linux host ได้
- **Snapshot / Rollback** — ย้อนสถานะเครื่องทั้งเครื่องได้
- **เหมาะกับ legacy system** ที่ต้องการ OS เก่า

### 👎 ข้อเสียของ VM

- **กินทรัพยากรมาก** — ต้องจอง RAM/CPU ล่วงหน้า
- **ขนาดใหญ่** — image หลาย GB
- **บูตช้า** — ใช้เวลาเป็นนาที
- **ซ้ำซ้อน** — รัน OS kernel หลายชุดบนเครื่องเดียว

### 💡 ในบริบท AI

การตั้ง VM สำหรับ deep learning ยุ่งยาก: ต้อง passthrough GPU (PCIe passthrough) ซึ่งซับซ้อน, สิ้นเปลือง RAM ที่ควรใช้โหลด dataset, และการก๊อป VM image ขนาด 30 GB ไปมาไม่สนุกเลย

---

## 3. Container คืออะไร

### 📦 นิยาม

**Container** คือหน่วยของซอฟต์แวร์ที่ห่อหุ้มแอปพลิเคชันพร้อม dependencies ทั้งหมดไว้ด้วยกัน โดย **ใช้ kernel ร่วมกับ Host OS** ไม่ต้องมี Guest OS ของตัวเอง

เปรียบเทียบ: VM = บ้านทั้งหลังพร้อมระบบไฟ-ประปาของตัวเอง | Container = ห้องในคอนโด ใช้ระบบส่วนกลางร่วมกันแต่มีประตูล็อกของตัวเอง

### 🏗️ สถาปัตยกรรม

```
┌─────────────────────────────────────────────┐
│   App A     │    App B     │     App C      │
├─────────────┼──────────────┼────────────────┤
│  Bins/Libs  │  Bins/Libs   │   Bins/Libs    │  ← เล็ก ไม่กี่ MB
├─────────────┴──────────────┴────────────────┤
│         Container Runtime (Docker)          │
├─────────────────────────────────────────────┤
│         Host OS (Shared Kernel)             │  ← ใช้ร่วมกัน!
├─────────────────────────────────────────────┤
│         Physical Hardware (Server)          │
└─────────────────────────────────────────────┘
```

### ⚙️ เทคโนโลยีเบื้องหลัง (Linux Kernel Features)

Container ไม่ใช่เวทมนตร์ แต่เป็นการนำฟีเจอร์ของ Linux kernel มาประกอบกัน:

| เทคโนโลยี | หน้าที่ |
|-----------|---------|
| **Namespaces** | แยกมุมมองของ process — PID, Network, Mount, User, IPC, UTS |
| **cgroups** | จำกัดทรัพยากร — CPU, memory, disk I/O ที่แต่ละคอนเทนเนอร์ใช้ได้ |
| **Union File System** | ซ้อนชั้นไฟล์ (layers) ทำให้ image แชร์กันได้ ประหยัดพื้นที่ |
| **Capabilities / seccomp** | จำกัดสิทธิ์ระดับ system call เพื่อความปลอดภัย |

### 👍 ข้อดีของ Container

- **เบาและเร็ว** — เริ่มทำงานในเสี้ยววินาที
- **ขนาดเล็ก** — image ตั้งแต่ 5 MB (alpine) ถึงไม่กี่ร้อย MB
- **Density สูง** — รันได้หลายสิบคอนเทนเนอร์บนเครื่องเดียว
- **Portable** — build ครั้งเดียว รันได้ทุกที่ที่มี container runtime
- **เข้ากับ CI/CD และ Kubernetes ได้ดีมาก**

### 👎 ข้อจำกัดของ Container

- **Isolation อ่อนกว่า VM** — ใช้ kernel ร่วมกัน หาก kernel มีช่องโหว่ก็เสี่ยงทั้งเครื่อง
- **ผูกกับ kernel ของ host** — container Linux รันบน Linux kernel เท่านั้น (บน Mac/Windows ใช้ VM เล็ก ๆ ซ่อนอยู่เบื้องหลัง)
- **ไม่เหมาะกับงานที่ต้องการ OS เต็มรูปแบบ** หรือ GUI หนัก ๆ

---

## 4. Container vs Virtual Machine

### 📊 ตารางเปรียบเทียบ

| หัวข้อ | 🐳 Container | 🖥️ Virtual Machine |
|--------|-------------|-------------------|
| **ระดับการจำลอง** | ระดับ OS (virtualize OS) | ระดับฮาร์ดแวร์ (virtualize hardware) |
| **Guest OS** | ไม่มี — ใช้ kernel ของ host | มี OS เต็มรูปแบบของตัวเอง |
| **ขนาด** | MB (5 MB – 2 GB) | GB (1 GB – 100 GB) |
| **เวลาเริ่มทำงาน** | มิลลิวินาที – วินาที | หลายสิบวินาที – นาที |
| **ใช้ทรัพยากร** | น้อย แชร์กันได้แบบยืดหยุ่น | มาก ต้องจองล่วงหน้า |
| **Isolation** | ระดับ process (อ่อนกว่า) | ระดับฮาร์ดแวร์ (แข็งแรงกว่า) |
| **ความปลอดภัย** | ดี แต่แชร์ kernel | ดีกว่า แยกขาดจากกัน |
| **รัน OS ต่างชนิด** | ไม่ได้ (Linux container ต้อง Linux kernel) | ได้ |
| **จำนวนต่อเครื่อง** | หลายสิบ–หลายร้อย | ไม่กี่ตัว |
| **การจัดการ GPU** | ง่าย ผ่าน NVIDIA Container Toolkit | ยาก ต้อง PCIe passthrough |
| **Portability** | สูงมาก | ปานกลาง (image ใหญ่) |
| **เหมาะกับ** | Microservices, ML serving, CI/CD | Legacy app, multi-OS, งานที่ต้องการความปลอดภัยสูงสุด |

### 🤝 ไม่ใช่คู่แข่ง — ใช้ร่วมกันได้

ในโลกจริง Cloud ทุกเจ้าใช้ทั้งสองอย่างซ้อนกัน:

```
Physical Server → VM (EC2 instance) → Docker Containers → Your ML App
```

VM ให้ความปลอดภัยระหว่างลูกค้าคนละราย ส่วน Container ให้ความเร็วและความยืดหยุ่นภายในของแต่ละราย

### 🧭 เลือกอย่างไร

- **เลือก Container** เมื่อต้องการ deploy บ่อย, สเกลเร็ว, ใช้ microservices, ทำ ML pipeline
- **เลือก VM** เมื่อต้องรัน OS ต่างชนิด, ต้องการ isolation ระดับสูงสุด, หรือรันระบบเก่าที่ย้ายไม่ได้

---

## 5. Image, Container, Registry

### 🧩 สามแนวคิดหลักที่ต้องแยกให้ออก

| แนวคิด | เปรียบเทียบ | คำอธิบาย |
|--------|------------|----------|
| **Image** | สูตรอาหาร / Class | แม่แบบแบบอ่านอย่างเดียว (read-only) ประกอบด้วย layers |
| **Container** | จานอาหารที่ปรุงเสร็จ / Object | instance ที่กำลังรันของ image เขียนข้อมูลได้ |
| **Registry** | ห้องสมุดสูตรอาหาร | ที่เก็บและแจกจ่าย image เช่น Docker Hub |

```
Dockerfile  ──build──▶  Image  ──run──▶  Container
                          │
                          ├──push──▶  Registry  ──pull──▶  เครื่องอื่น
```

### 🎂 Image Layers — หัวใจของความเร็ว

Image ประกอบด้วยชั้น (layer) ซ้อนกัน แต่ละคำสั่งใน Dockerfile สร้างหนึ่ง layer

```
┌──────────────────────────┐
│ Layer 5: COPY app code   │  ← เปลี่ยนบ่อย
├──────────────────────────┤
│ Layer 4: pip install     │  ← เปลี่ยนนาน ๆ ครั้ง
├──────────────────────────┤
│ Layer 3: apt-get install │
├──────────────────────────┤
│ Layer 2: Python runtime  │
├──────────────────────────┤
│ Layer 1: Base OS (slim)  │  ← แทบไม่เปลี่ยน
└──────────────────────────┘
```

**ประโยชน์:**
- **Caching** — build ใหม่จะใช้ layer เดิมที่ไม่เปลี่ยน ทำให้เร็วมาก
- **แชร์ layer** — 10 image ที่ใช้ `python:3.11-slim` เหมือนกัน เก็บ base layer แค่ชุดเดียว
- **โอนเฉพาะส่วนต่าง** — `docker pull` ดาวน์โหลดเฉพาะ layer ที่ยังไม่มี

> ⚠️ **สำคัญ:** เรียงคำสั่งใน Dockerfile จาก "เปลี่ยนน้อย" ไป "เปลี่ยนบ่อย" เสมอ เพื่อใช้ cache ให้คุ้ม

### 🏷️ Image Tags

```bash
# โครงสร้าง: [registry]/[namespace]/[repository]:[tag]
python:3.11-slim
nvidia/cuda:12.2.0-runtime-ubuntu22.04
ghcr.io/myorg/ml-api:v1.2.3
myregistry.azurecr.io/fraud-model:latest
```

- `latest` **ไม่ได้แปลว่าใหม่ล่าสุดเสมอ** — เป็นแค่ tag เริ่มต้น **อย่าใช้ใน production**
- ระบุเวอร์ชันชัดเจนเสมอ เช่น `v1.2.3` หรือใช้ digest `@sha256:...` เพื่อความแน่นอนสูงสุด

### 🏪 Registry ยอดนิยม

| Registry | ลักษณะ |
|----------|--------|
| **Docker Hub** | ค่าเริ่มต้น มี official images จำนวนมาก มี rate limit สำหรับ free tier |
| **GitHub Container Registry (ghcr.io)** | ผูกกับ GitHub Actions ได้ดี |
| **NVIDIA NGC** | image สำหรับ AI/HPC ที่ปรับแต่งมาแล้ว — PyTorch, TensorFlow, Triton |
| **AWS ECR / Google Artifact Registry / Azure ACR** | สำหรับใช้ภายในองค์กรบน cloud |
| **Harbor** | self-hosted registry พร้อมสแกนช่องโหว่ |

---

## 6. Core Commands & Dockerfiles

หมวดนี้คือส่วนลงมือปฏิบัติ เราจะเรียน 2 เรื่องที่ใช้ทุกวัน:

- **หัวข้อ 7** — คำสั่ง Docker ที่ต้องจำให้ได้
- **หัวข้อ 8** — การเขียน Dockerfile สำหรับงาน ML

### 🎯 เป้าหมายของหมวดนี้

เมื่อจบหมวดนี้ คุณจะสามารถ:
1. ดึง image มารัน หยุด ลบ และดู log ได้คล่อง
2. เข้าไปใน shell ของคอนเทนเนอร์เพื่อ debug ได้
3. เขียน Dockerfile ห่อโปรเจกต์ Python ของตัวเองได้
4. Build image และ push ขึ้น registry ได้

### ✅ ตรวจสอบว่าติดตั้ง Docker เรียบร้อยแล้ว

```bash
docker --version          # ดูเวอร์ชัน
docker info               # ดูรายละเอียดระบบ
docker run hello-world    # ทดสอบรันคอนเทนเนอร์แรก
```

ถ้าเห็นข้อความ `Hello from Docker!` แปลว่าพร้อมเรียนต่อได้เลย 🎉

---

## 7. คำสั่งหลักของ Docker

### 🔄 วงจรชีวิตของคอนเทนเนอร์

```
docker pull → docker run → docker stop → docker start → docker rm
```

### 📥 จัดการ Image

```bash
# ดาวน์โหลด image จาก registry
docker pull python:3.11-slim

# ดูรายการ image ในเครื่อง
docker images
docker image ls

# ลบ image
docker rmi python:3.11-slim

# ค้นหา image บน Docker Hub
docker search pytorch

# ดูประวัติ layer ของ image
docker history myapp:v1

# ติด tag ใหม่ให้ image
docker tag myapp:v1 myregistry.com/myapp:v1

# ส่ง image ขึ้น registry
docker login
docker push myregistry.com/myapp:v1
```

### ▶️ รันคอนเทนเนอร์

```bash
# รันแบบพื้นฐาน
docker run nginx

# รันเบื้องหลัง (detached) + ตั้งชื่อ + แมป port
docker run -d --name web -p 8080:80 nginx

# รันแบบโต้ตอบ เข้า shell
docker run -it python:3.11-slim bash

# รันแล้วลบอัตโนมัติเมื่อจบ (เหมาะกับงานทดสอบ)
docker run --rm -it python:3.11-slim python -c "print('hi')"

# ตั้งค่า environment variable
docker run -e MODEL_PATH=/models/v1 -e LOG_LEVEL=debug myapp

# เมาต์โฟลเดอร์จากเครื่องเข้าไป
docker run -v $(pwd)/data:/app/data myapp

# จำกัดทรัพยากร
docker run --cpus="2" --memory="4g" myapp

# ใช้ GPU ทั้งหมด
docker run --gpus all nvidia/cuda:12.2.0-base-ubuntu22.04 nvidia-smi
```

### 🎛️ Flag ที่ใช้บ่อยที่สุด

| Flag | ความหมาย |
|------|----------|
| `-d` | รันเบื้องหลัง (detached) |
| `-it` | โต้ตอบได้ + มี terminal (ใช้คู่กันเสมอ) |
| `--rm` | ลบคอนเทนเนอร์อัตโนมัติเมื่อหยุด |
| `-p host:container` | แมปพอร์ต |
| `-v host:container` | เมาต์ volume / bind mount |
| `-e KEY=VALUE` | ตั้งค่า environment variable |
| `--name` | ตั้งชื่อคอนเทนเนอร์ |
| `--network` | เลือกเครือข่าย |
| `--gpus all` | เปิดใช้ GPU |
| `-w /path` | ตั้ง working directory |

### 🔍 ตรวจสอบและ Debug

```bash
# ดูคอนเทนเนอร์ที่กำลังรัน
docker ps

# ดูทั้งหมดรวมที่หยุดแล้ว
docker ps -a

# ดู log
docker logs web
docker logs -f web              # ตามแบบ real-time
docker logs --tail 100 web      # ดู 100 บรรทัดล่าสุด

# เข้าไปใน shell ของคอนเทนเนอร์ที่รันอยู่ (สำคัญมากสำหรับ debug)
docker exec -it web bash
docker exec -it web sh          # ถ้า image ไม่มี bash

# รันคำสั่งเดียวในคอนเทนเนอร์
docker exec web ls /app

# ดูข้อมูลละเอียดแบบ JSON
docker inspect web

# ดูการใช้ทรัพยากรแบบ real-time
docker stats

# ดู process ในคอนเทนเนอร์
docker top web

# คัดลอกไฟล์เข้า-ออก
docker cp web:/app/output.csv ./output.csv
docker cp ./config.yaml web:/app/config.yaml
```

### ⏹️ หยุดและลบ

```bash
docker stop web              # หยุดอย่างนุ่มนวล (SIGTERM)
docker kill web              # บังคับหยุดทันที (SIGKILL)
docker start web             # เริ่มใหม่
docker restart web           # รีสตาร์ท
docker rm web                # ลบคอนเทนเนอร์
docker rm -f web             # บังคับลบทั้งที่ยังรันอยู่
```

### 🧹 ทำความสะอาด (สำคัญมาก — Docker กินพื้นที่เยอะ)

```bash
docker system df             # ดูว่า Docker ใช้พื้นที่เท่าไร
docker container prune       # ลบคอนเทนเนอร์ที่หยุดแล้วทั้งหมด
docker image prune           # ลบ dangling images
docker image prune -a        # ลบ image ที่ไม่มีคอนเทนเนอร์ใช้
docker volume prune          # ลบ volume ที่ไม่ได้ใช้
docker system prune -a       # ⚠️ ล้างทุกอย่างที่ไม่ได้ใช้
```

---

## 8. การเขียน Dockerfile

### 📄 Dockerfile คืออะไร

ไฟล์ข้อความที่บอก Docker ว่าจะสร้าง image อย่างไร — เปรียบเหมือน "สูตรอาหารที่เขียนเป็นโค้ด" เก็บใน Git ได้ ทบทวนได้ ย้อนกลับได้

### 🔤 คำสั่งหลักใน Dockerfile

| คำสั่ง | หน้าที่ | ตัวอย่าง |
|--------|---------|----------|
| `FROM` | กำหนด base image (ต้องมาก่อนเสมอ) | `FROM python:3.11-slim` |
| `WORKDIR` | ตั้ง working directory | `WORKDIR /app` |
| `COPY` | คัดลอกไฟล์จาก host เข้า image | `COPY . /app` |
| `ADD` | เหมือน COPY แต่แตกไฟล์ tar/ดึง URL ได้ | `ADD model.tar.gz /models/` |
| `RUN` | รันคำสั่งตอน **build** | `RUN pip install -r requirements.txt` |
| `ENV` | ตั้ง environment variable ถาวร | `ENV PYTHONUNBUFFERED=1` |
| `ARG` | ตัวแปรที่ใช้เฉพาะตอน build | `ARG VERSION=1.0` |
| `EXPOSE` | ประกาศพอร์ตที่ใช้ (เป็นเอกสาร) | `EXPOSE 8000` |
| `VOLUME` | ประกาศจุดเมาต์ | `VOLUME /data` |
| `USER` | เปลี่ยนผู้ใช้ที่รัน | `USER appuser` |
| `HEALTHCHECK` | ตรวจสุขภาพคอนเทนเนอร์ | ดูตัวอย่างด้านล่าง |
| `CMD` | คำสั่งเริ่มต้นตอน **run** (แทนที่ได้) | `CMD ["python", "app.py"]` |
| `ENTRYPOINT` | คำสั่งหลักที่แทนที่ยาก | `ENTRYPOINT ["python"]` |

### ⚖️ CMD vs ENTRYPOINT

```dockerfile
# แบบ CMD — แทนที่ได้ง่าย
CMD ["python", "app.py"]
# docker run myapp python train.py  → รัน train.py แทน

# แบบ ENTRYPOINT + CMD — ENTRYPOINT คงที่ CMD เป็นค่า default ของ argument
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8000"]
# docker run myapp --port 9000  → python app.py --port 9000
```

> 💡 ใช้ **exec form** `["cmd", "arg"]` เสมอ ไม่ใช่ shell form `cmd arg` เพราะ exec form ส่งสัญญาณ SIGTERM ถึง process ได้ถูกต้อง ทำให้ปิดตัวอย่างนุ่มนวล

### 🐍 ตัวอย่างที่ 1: Dockerfile สำหรับ ML API พื้นฐาน

```dockerfile
FROM python:3.11-slim

# ป้องกัน Python เขียนไฟล์ .pyc และบังคับให้ log ออกทันที
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

# ติดตั้ง system dependencies (ถ้าจำเป็น)
RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
    && rm -rf /var/lib/apt/lists/*

# ⭐ คัดลอก requirements ก่อน เพื่อใช้ cache ให้คุ้ม
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# คัดลอกโค้ดทีหลัง (เปลี่ยนบ่อยที่สุด)
COPY . .

# สร้าง user ที่ไม่ใช่ root เพื่อความปลอดภัย
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')"

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

> ⚠️ **จุดที่คนพลาดบ่อย:** ต้องใช้ `--host 0.0.0.0` ไม่ใช่ `127.0.0.1` มิฉะนั้นจะเข้าถึงจากนอกคอนเทนเนอร์ไม่ได้

### 🏗️ ตัวอย่างที่ 2: Multi-stage Build (ลดขนาด image ได้มาก)

```dockerfile
# ---------- Stage 1: Builder ----------
FROM python:3.11 AS builder

WORKDIR /build
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# ---------- Stage 2: Runtime ----------
FROM python:3.11-slim

ENV PYTHONUNBUFFERED=1 \
    PATH=/home/appuser/.local/bin:$PATH

RUN useradd -m -u 1000 appuser
USER appuser
WORKDIR /app

# คัดลอกเฉพาะ package ที่ติดตั้งแล้ว ไม่เอา build tools มาด้วย
COPY --from=builder /root/.local /home/appuser/.local
COPY --chown=appuser:appuser . .

EXPOSE 8000
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

**ผลลัพธ์:** จาก ~1.2 GB เหลือ ~250 MB

### 📝 .dockerignore — อย่าลืมไฟล์นี้เด็ดขาด

```gitignore
# Version control
.git
.gitignore

# Python
__pycache__/
*.py[cod]
.venv/
venv/
.pytest_cache/
.mypy_cache/

# Notebooks & experiments
.ipynb_checkpoints/
notebooks/scratch/

# ข้อมูลขนาดใหญ่ — ใช้ volume แทน
data/
datasets/
*.csv
*.parquet

# โมเดล — ควรใช้ model registry
models/*.pt
models/*.h5
mlruns/
wandb/

# Secrets
.env
*.pem
credentials.json

# IDE & OS
.vscode/
.idea/
.DS_Store
```

### 🏅 Best Practices สำหรับ Dockerfile

1. **ใช้ base image ที่เล็กที่สุดเท่าที่ทำได้** — `slim` > `full`, ระวัง `alpine` กับ Python (มีปัญหากับ numpy/scipy เพราะใช้ musl libc)
2. **ตรึงเวอร์ชันเสมอ** — `python:3.11.8-slim` ไม่ใช่ `python:latest`
3. **เรียงจากเปลี่ยนน้อยไปเปลี่ยนบ่อย** — เพื่อใช้ cache
4. **รวมคำสั่ง RUN ด้วย `&&`** — ลดจำนวน layer
5. **ล้าง cache ใน layer เดียวกัน** — `rm -rf /var/lib/apt/lists/*`
6. **ใช้ `--no-cache-dir` กับ pip**
7. **ไม่รันด้วย root** — สร้าง non-root user
8. **ห้ามใส่ secret ใน Dockerfile** — ใช้ env var หรือ secret manager (secret ที่ใส่ใน layer จะอยู่ใน image ตลอดไป แม้จะลบทีหลัง)
9. **ใช้ multi-stage build** สำหรับ production
10. **ใส่ HEALTHCHECK** เพื่อให้ orchestrator รู้สถานะ

### 🔨 คำสั่ง Build

```bash
# build พื้นฐาน
docker build -t myapp:v1 .

# ระบุ Dockerfile คนละชื่อ
docker build -f Dockerfile.gpu -t myapp:gpu .

# ส่ง build argument
docker build --build-arg VERSION=2.0 -t myapp:v2 .

# build โดยไม่ใช้ cache
docker build --no-cache -t myapp:v1 .

# build สำหรับหลาย platform
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:v1 .
```

---

## 9. Volumes & Networking

หมวดนี้ตอบคำถามสำคัญ 2 ข้อที่ทุกคนเจอหลังรันคอนเทนเนอร์ได้แล้ว:

- **"ทำไมข้อมูลหายไปเมื่อลบคอนเทนเนอร์"** → หัวข้อ 10: Volumes
- **"ทำไมคอนเทนเนอร์คุยกันไม่ได้ / เข้าเว็บไม่ได้"** → หัวข้อ 11: Networking

### 🎯 ทำไมสองเรื่องนี้สำคัญกับงาน AI

- **Dataset ขนาดหลาย GB** ไม่ควรอยู่ใน image — ต้องเมาต์เข้าไป
- **Checkpoint ระหว่างเทรน** ต้องรอดแม้คอนเทนเนอร์พัง
- **ML API ต้องคุยกับ database, Redis, vector store** — ต้องเข้าใจเครือข่าย
- **Jupyter notebook** ต้องเข้าถึงจากเบราว์เซอร์บนเครื่องเรา — ต้อง map port

---

## 10. Volumes และการเก็บข้อมูลถาวร

### 💥 ปัญหา: คอนเทนเนอร์ไม่มีความทรงจำ

ข้อมูลที่เขียนในคอนเทนเนอร์จะอยู่ใน **writable layer** ซึ่ง **หายไปทันทีที่ลบคอนเทนเนอร์**

```bash
docker run -it --name test python:3.11-slim bash
# ในคอนเทนเนอร์: echo "ผลการเทรน" > /result.txt แล้ว exit
docker rm test    # ← ไฟล์หายหมด 😱
```

### 🗂️ สามวิธีเก็บข้อมูล

| วิธี | ตำแหน่งจัดเก็บ | เหมาะกับ |
|------|----------------|----------|
| **Named Volume** | Docker จัดการเอง (`/var/lib/docker/volumes/`) | ข้อมูล production, database, model artifacts |
| **Bind Mount** | โฟลเดอร์บนเครื่อง host | พัฒนาโค้ด, dataset, ดูผลลัพธ์ |
| **tmpfs Mount** | RAM เท่านั้น | ข้อมูลชั่วคราว, secret |

### 📦 Named Volume

```bash
# สร้าง volume
docker volume create ml-models

# ดูรายการ
docker volume ls

# ดูรายละเอียด
docker volume inspect ml-models

# ใช้งาน
docker run -v ml-models:/app/models myapp

# แบบ --mount (ชัดเจนกว่า แนะนำสำหรับ production)
docker run --mount type=volume,source=ml-models,target=/app/models myapp

# ลบ
docker volume rm ml-models
```

**ข้อดี:** Docker จัดการให้, ย้ายเครื่องได้, backup ง่าย, ทำงานเหมือนกันทุก OS

### 🔗 Bind Mount

```bash
# เมาต์โฟลเดอร์ปัจจุบันเข้าไป (Linux/macOS)
docker run -v $(pwd):/app myapp

# Windows PowerShell
docker run -v ${PWD}:/app myapp

# เมาต์ dataset แบบอ่านอย่างเดียว (ปลอดภัยกว่า)
docker run -v /home/user/datasets:/data:ro myapp

# แบบ --mount
docker run --mount type=bind,source="$(pwd)",target=/app myapp
```

**ข้อดี:** แก้โค้ดบนเครื่อง เห็นผลในคอนเทนเนอร์ทันที — เหมาะกับการพัฒนามาก
**ข้อเสีย:** ผูกกับ path ของเครื่อง, บน macOS/Windows อาจช้ากว่า

### 🧪 ตัวอย่างจริงสำหรับงาน ML

**1. พัฒนาโค้ดพร้อม hot reload**

```bash
docker run -it --rm \
  -v $(pwd)/src:/app/src \
  -v $(pwd)/notebooks:/app/notebooks \
  -p 8888:8888 \
  jupyter/scipy-notebook
```

**2. เทรนโมเดลพร้อมเก็บ checkpoint**

```bash
docker run --gpus all \
  -v /mnt/datasets:/data:ro \
  -v ml-checkpoints:/app/checkpoints \
  -v $(pwd)/logs:/app/logs \
  my-training-image \
  python train.py --epochs 100
```

**3. Serve โมเดลจาก volume**

```bash
docker run -d -p 8000:8000 \
  -v ml-models:/models:ro \
  -e MODEL_PATH=/models/fraud_v3.pkl \
  ml-api:v1
```

### 💾 สำรองและกู้คืน Volume

```bash
# Backup volume เป็นไฟล์ tar
docker run --rm \
  -v ml-models:/source:ro \
  -v $(pwd):/backup \
  alpine tar czf /backup/models-backup.tar.gz -C /source .

# Restore
docker run --rm \
  -v ml-models:/target \
  -v $(pwd):/backup \
  alpine sh -c "cd /target && tar xzf /backup/models-backup.tar.gz"
```

### ⚠️ ข้อควรระวัง

- **สิทธิ์ไฟล์ (Permissions)** — ไฟล์ที่คอนเทนเนอร์สร้างอาจเป็นของ root ใช้ `--user $(id -u):$(id -g)` ช่วยได้
- **อย่าใส่ dataset ใหญ่ใน image** — image จะบวมและ build ช้ามาก
- **`docker volume prune` ลบข้อมูลถาวร** — ระวังให้มาก
- **บน macOS/Windows bind mount ช้า** — ถ้าต้องการ I/O เร็วให้ใช้ named volume

---

## 11. Docker Networking และ Port Mapping

### 🌐 Network Drivers

| Driver | คำอธิบาย | การใช้งาน |
|--------|----------|-----------|
| **bridge** | ค่าเริ่มต้น สร้างเครือข่ายเสมือนภายในเครื่อง | งานทั่วไป, multi-container app |
| **host** | ใช้ network stack ของ host โดยตรง ไม่มี isolation | ต้องการประสิทธิภาพสูงสุด (Linux เท่านั้น) |
| **none** | ไม่มีเครือข่ายเลย | งานที่ต้องการความปลอดภัยสูง, batch job |
| **overlay** | เชื่อมคอนเทนเนอร์ข้ามหลายเครื่อง | Docker Swarm, cluster |
| **macvlan** | ให้คอนเทนเนอร์มี MAC address ของตัวเอง | เชื่อมกับเครือข่ายเดิมขององค์กร |

### 🔌 Port Mapping

```bash
# รูปแบบ: -p [host_port]:[container_port]
docker run -p 8080:8000 myapp
#            ▲     ▲
#            │     └── พอร์ตในคอนเทนเนอร์ (ที่แอปฟัง)
#            └──────── พอร์ตบนเครื่องเรา (ที่เราเข้าถึง)

# เข้าใช้งานที่ http://localhost:8080

# ผูกกับ IP เฉพาะ (ปลอดภัยกว่า - เข้าได้แค่จากเครื่องตัวเอง)
docker run -p 127.0.0.1:8080:8000 myapp

# หลายพอร์ต
docker run -p 8000:8000 -p 9090:9090 myapp

# สุ่มพอร์ตบน host
docker run -P myapp
docker port myapp        # ดูว่าได้พอร์ตอะไร

# ระบุ protocol
docker run -p 5353:5353/udp myapp
```

### 🔧 จัดการ Network

```bash
# ดูรายการเครือข่าย
docker network ls

# สร้างเครือข่ายของตัวเอง (แนะนำ)
docker network create ml-network

# รันคอนเทนเนอร์ในเครือข่ายนั้น
docker run -d --name postgres --network ml-network postgres:16
docker run -d --name api --network ml-network -p 8000:8000 ml-api

# เชื่อมคอนเทนเนอร์ที่รันอยู่แล้วเข้าเครือข่าย
docker network connect ml-network existing-container

# ตัดการเชื่อมต่อ
docker network disconnect ml-network existing-container

# ดูรายละเอียด
docker network inspect ml-network

# ลบ
docker network rm ml-network
```

### 🏷️ Service Discovery — จุดสำคัญที่สุด

**เมื่ออยู่ใน user-defined network เดียวกัน คอนเทนเนอร์เรียกกันด้วย "ชื่อคอนเทนเนอร์" ได้เลย**

```python
# ในโค้ด Python ของ API
# ❌ ผิด — localhost ในคอนเทนเนอร์หมายถึงตัวคอนเทนเนอร์เอง
DATABASE_URL = "postgresql://user:pass@localhost:5432/mldb"

# ✅ ถูก — ใช้ชื่อคอนเทนเนอร์เป็น hostname
DATABASE_URL = "postgresql://user:pass@postgres:5432/mldb"
REDIS_URL = "redis://redis:6379"
VECTOR_DB = "http://qdrant:6333"
```

> 💡 **หมายเหตุ:** default bridge network ไม่มี DNS resolution แบบนี้ ต้องสร้าง user-defined network เองเสมอ

### 🖥️ เข้าถึงเครื่อง Host จากในคอนเทนเนอร์

```bash
# Docker Desktop (Mac/Windows) — ใช้ได้เลย
host.docker.internal

# Linux — ต้องเพิ่ม flag
docker run --add-host=host.docker.internal:host-gateway myapp
```

### 🐞 Debug ปัญหาเครือข่าย

```bash
# ตรวจว่า port map ถูกไหม
docker ps
docker port myapp

# เข้าไปทดสอบจากในคอนเทนเนอร์
docker exec -it api bash
  ping postgres
  curl http://postgres:5432
  nslookup redis

# ใช้ image เครื่องมือเครือข่าย
docker run --rm -it --network ml-network nicolaka/netshoot
```

### ❗ ปัญหาที่เจอบ่อย

| อาการ | สาเหตุ | วิธีแก้ |
|-------|--------|---------|
| เข้า `localhost:8000` ไม่ได้ | แอปฟังที่ `127.0.0.1` ในคอนเทนเนอร์ | ตั้งให้ฟังที่ `0.0.0.0` |
| คอนเทนเนอร์คุยกันไม่ได้ | อยู่คนละ network หรือใช้ default bridge | สร้าง user-defined network |
| `port is already allocated` | พอร์ตบน host ถูกใช้แล้ว | เปลี่ยนพอร์ต host หรือหยุดตัวที่ใช้อยู่ |
| DB connection refused | DB ยังบูตไม่เสร็จ | ใส่ healthcheck + retry logic |

---

## 12. Compose & AI-Specific Tips

หมวดนี้ยกระดับจาก "รันคอนเทนเนอร์เดียว" ไปสู่ "ระบบจริง"

- **หัวข้อ 13** — Docker Compose จัดการหลายคอนเทนเนอร์พร้อมกัน
- **หัวข้อ 14** — เทคนิคเฉพาะสำหรับงาน AI: GPU, image ขนาดใหญ่, การเทรน

### 🤔 ทำไมต้องใช้ Compose

ลองนึกภาพระบบ ML จริงหนึ่งระบบ:

```
FastAPI (โมเดล) + PostgreSQL (metadata) + Redis (cache)
+ MLflow (tracking) + MinIO (เก็บ artifact) + Qdrant (vector DB)
```

ถ้ารันด้วย `docker run` ทีละตัว = 6 คำสั่งยาว ๆ ที่ต้องจำลำดับและ flag ทั้งหมด
ด้วย Compose = `docker compose up` คำสั่งเดียว ✨

---

## 13. Docker Compose: แอปแบบหลายคอนเทนเนอร์

### 📘 Docker Compose คืออะไร

เครื่องมือที่ให้เรานิยามระบบหลายคอนเทนเนอร์ในไฟล์ YAML ไฟล์เดียว แล้วสั่งงานทั้งระบบพร้อมกัน

### 📄 โครงสร้างไฟล์ `docker-compose.yml`

```yaml
services:          # คอนเทนเนอร์แต่ละตัว
volumes:           # volume ที่ใช้ร่วมกัน
networks:          # เครือข่าย
```

### 🧱 ตัวอย่างที่ 1: ML API + Database + Cache

```yaml
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: ml-api
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://mluser:mlpass@postgres:5432/mldb
      - REDIS_URL=redis://redis:6379
      - MODEL_PATH=/models/model.pkl
    volumes:
      - ./src:/app/src            # hot reload ตอนพัฒนา
      - ml-models:/models:ro
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped
    networks:
      - ml-net

  postgres:
    image: postgres:16-alpine
    container_name: ml-postgres
    environment:
      POSTGRES_USER: mluser
      POSTGRES_PASSWORD: mlpass
      POSTGRES_DB: mldb
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U mluser -d mldb"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - ml-net

  redis:
    image: redis:7-alpine
    container_name: ml-redis
    command: redis-server --appendonly yes
    volumes:
      - redisdata:/data
    networks:
      - ml-net

volumes:
  pgdata:
  redisdata:
  ml-models:

networks:
  ml-net:
    driver: bridge
```

### 🧪 ตัวอย่างที่ 2: MLflow Tracking Stack

```yaml
services:
  mlflow:
    image: ghcr.io/mlflow/mlflow:v2.14.1
    ports:
      - "5000:5000"
    environment:
      - MLFLOW_S3_ENDPOINT_URL=http://minio:9000
      - AWS_ACCESS_KEY_ID=minioadmin
      - AWS_SECRET_ACCESS_KEY=minioadmin
    command: >
      mlflow server
      --host 0.0.0.0
      --backend-store-uri postgresql://mluser:mlpass@postgres:5432/mlflow
      --default-artifact-root s3://mlflow/
    depends_on:
      - postgres
      - minio

  minio:
    image: minio/minio:latest
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    command: server /data --console-address ":9001"
    volumes:
      - miniodata:/data

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: mluser
      POSTGRES_PASSWORD: mlpass
      POSTGRES_DB: mlflow
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  miniodata:
  pgdata:
```

### ⌨️ คำสั่ง Compose ที่ใช้บ่อย

```bash
docker compose up                  # เริ่มทั้งหมด (เห็น log)
docker compose up -d               # เริ่มเบื้องหลัง
docker compose up --build          # build ใหม่ก่อนเริ่ม
docker compose up -d api postgres  # เริ่มเฉพาะบาง service

docker compose ps                  # ดูสถานะ
docker compose logs -f             # ดู log ทั้งหมด
docker compose logs -f api         # ดู log เฉพาะ service

docker compose exec api bash       # เข้า shell
docker compose run --rm api pytest # รันคำสั่งครั้งเดียว

docker compose stop                # หยุด (ไม่ลบ)
docker compose start               # เริ่มต่อ
docker compose restart api         # รีสตาร์ท service

docker compose down                # หยุดและลบคอนเทนเนอร์ + network
docker compose down -v             # ⚠️ ลบ volume ด้วย (ข้อมูลหาย)

docker compose config              # ตรวจสอบไฟล์ YAML
docker compose build --no-cache    # build ใหม่หมด
```

### 🔐 จัดการ Environment Variables

**ไฟล์ `.env`** (อย่าลืมใส่ใน `.gitignore`)

```env
POSTGRES_USER=mluser
POSTGRES_PASSWORD=super-secret
MODEL_VERSION=v3
API_PORT=8000
```

**เรียกใช้ใน compose:**

```yaml
services:
  api:
    ports:
      - "${API_PORT}:8000"
    environment:
      - MODEL_VERSION=${MODEL_VERSION}
    env_file:
      - .env
```

### 📚 แยกไฟล์สำหรับ Dev / Production

```bash
# base + override สำหรับ production
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

```yaml
# docker-compose.prod.yml
services:
  api:
    volumes: []                    # ไม่ต้อง hot reload
    environment:
      - LOG_LEVEL=warning
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: "2"
          memory: 4G
    restart: always
```

---

## 14. Docker สำหรับงาน AI Workloads

### 🎮 การใช้ GPU ในคอนเทนเนอร์

**ขั้นตอนเตรียมเครื่อง (Linux host):**

1. ติดตั้ง NVIDIA driver บน host
2. ติดตั้ง [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)
3. รีสตาร์ท Docker daemon

```bash
# ทดสอบว่า GPU ใช้ได้
docker run --rm --gpus all nvidia/cuda:12.2.0-base-ubuntu22.04 nvidia-smi

# ใช้ GPU ทั้งหมด
docker run --gpus all my-training-image

# ใช้ GPU 2 ตัว
docker run --gpus 2 my-training-image

# เลือก GPU เฉพาะใบ
docker run --gpus '"device=0,2"' my-training-image
```

**ใน Docker Compose:**

```yaml
services:
  trainer:
    image: my-training:latest
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
```

### 🔥 Dockerfile สำหรับ PyTorch + CUDA

```dockerfile
FROM nvidia/cuda:12.2.0-cudnn8-runtime-ubuntu22.04

ENV DEBIAN_FRONTEND=noninteractive \
    PYTHONUNBUFFERED=1

RUN apt-get update && apt-get install -y --no-install-recommends \
        python3.11 python3-pip \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY requirements.txt .
RUN pip3 install --no-cache-dir \
        torch torchvision --index-url https://download.pytorch.org/whl/cu121 \
    && pip3 install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python3", "train.py"]
```

> 💡 **ทางลัด:** ใช้ image สำเร็จรูปจาก NGC เช่น `nvcr.io/nvidia/pytorch:24.05-py3` ซึ่งปรับแต่งมาแล้วและมี library ครบ

### ⚡ เทคนิคสำคัญเฉพาะงาน AI

**1. อย่าใส่โมเดลและ dataset ลงใน image**

```dockerfile
# ❌ แย่ — image บวมเป็น 10 GB
COPY models/llama-7b.bin /models/

# ✅ ดี — เมาต์ตอน run หรือดาวน์โหลดตอนเริ่ม
VOLUME /models
```

**2. ใช้ shared memory ให้พอสำหรับ PyTorch DataLoader**

```bash
# ป้องกัน error "DataLoader worker killed"
docker run --shm-size=8g --gpus all my-training-image
```

```yaml
services:
  trainer:
    shm_size: '8gb'
```

**3. แคช HuggingFace models ไว้นอกคอนเทนเนอร์**

```bash
docker run \
  -v hf-cache:/root/.cache/huggingface \
  -e HF_HOME=/root/.cache/huggingface \
  my-llm-app
```

**4. ตรึงเวอร์ชันทุกอย่างเพื่อ reproducibility**

```txt
# requirements.txt
torch==2.3.0
transformers==4.41.2
numpy==1.26.4
scikit-learn==1.5.0
```

**5. แยก image สำหรับ train และ serve**

| Image | ขนาด | มีอะไร |
|-------|------|--------|
| `myapp:train` | ~8 GB | CUDA, PyTorch เต็ม, jupyter, dev tools |
| `myapp:serve` | ~400 MB | runtime เท่านั้น + onnxruntime หรือ torch CPU |

**6. ใช้ BuildKit cache mount เร่ง pip install**

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.11-slim
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt
```

**7. ตั้ง resource limits ป้องกัน OOM**

```bash
docker run --memory="16g" --cpus="8" --memory-swap="16g" my-training-image
```

**8. Training job ที่รันนานควรตั้ง restart policy**

```yaml
services:
  trainer:
    restart: on-failure:3
```

### 🚀 ตัวเลือกสำหรับ Model Serving

| เครื่องมือ | เหมาะกับ |
|-----------|----------|
| **FastAPI + Uvicorn** | โมเดลทั่วไป เริ่มต้นง่ายที่สุด |
| **NVIDIA Triton** | หลายโมเดล หลาย framework ประสิทธิภาพสูง |
| **TorchServe** | เฉพาะ PyTorch |
| **TensorFlow Serving** | เฉพาะ TensorFlow |
| **vLLM** | LLM inference ประสิทธิภาพสูง |
| **Ollama** | รัน LLM ในเครื่องแบบง่าย ๆ |
| **BentoML** | ครบวงจร build + deploy |

### 🔒 Security สำหรับงาน AI

```bash
# สแกนช่องโหว่ใน image
docker scout cves myapp:v1
trivy image myapp:v1

# รันแบบ read-only filesystem
docker run --read-only --tmpfs /tmp myapp

# ไม่ให้เพิ่มสิทธิ์
docker run --security-opt=no-new-privileges myapp
```

> ⚠️ **ห้ามใส่ API key ใน image เด็ดขาด** — ใช้ env var, Docker secrets หรือ cloud secret manager

---

## 15. Cheat Sheet & Final Challenge

หมวดสุดท้าย! คุณผ่านทฤษฎีและปฏิบัติมาครบแล้ว ตอนนี้เหลืออีก 2 ส่วน:

- **หัวข้อ 16** — Cheat Sheet สำหรับเปิดดูเวลาทำงานจริง
- **หัวข้อ 17** — โจทย์ท้าทาย: สร้างและ deploy ML service ด้วยตัวเอง

### ✅ ทบทวนสิ่งที่เรียนมา

- [ ] เข้าใจความต่างระหว่าง VM กับ Container
- [ ] แยก Image / Container / Registry ได้
- [ ] รัน หยุด ลบ และ debug คอนเทนเนอร์ได้
- [ ] เขียน Dockerfile สำหรับโปรเจกต์ Python ได้
- [ ] ใช้ Volume เก็บข้อมูลถาวรได้
- [ ] เข้าใจ port mapping และ service discovery
- [ ] เขียน docker-compose.yml สำหรับระบบหลายคอนเทนเนอร์ได้
- [ ] รู้เทคนิคเฉพาะสำหรับงาน AI (GPU, shm-size, cache)

---

## 16. Docker Cheat Sheet

### 🖼️ Images

```bash
docker pull IMAGE                      # ดาวน์โหลด image
docker images                          # ดูรายการ image
docker build -t NAME:TAG .             # build จาก Dockerfile
docker tag SRC TARGET                  # ติด tag ใหม่
docker push NAME:TAG                   # ส่งขึ้น registry
docker rmi IMAGE                       # ลบ image
docker history IMAGE                   # ดู layer
docker save -o out.tar IMAGE           # export เป็นไฟล์
docker load -i out.tar                 # import จากไฟล์
```

### 📦 Containers

```bash
docker run IMAGE                       # รัน
docker run -d --name N -p 8080:80 IMG  # รันเบื้องหลัง + ตั้งชื่อ + map port
docker run -it --rm IMG bash           # เข้า shell แล้วลบทิ้งเมื่อออก
docker ps                              # ดูที่กำลังรัน
docker ps -a                           # ดูทั้งหมด
docker stop / start / restart NAME     # หยุด / เริ่ม / รีสตาร์ท
docker rm NAME                         # ลบ
docker rm -f NAME                      # บังคับลบ
docker rename OLD NEW                  # เปลี่ยนชื่อ
```

### 🔍 Debug

```bash
docker logs -f NAME                    # ดู log แบบ real-time
docker logs --tail 100 NAME            # ดู 100 บรรทัดล่าสุด
docker exec -it NAME bash              # เข้า shell
docker inspect NAME                    # ดูรายละเอียด JSON
docker stats                           # ดูการใช้ทรัพยากร
docker top NAME                        # ดู process
docker cp NAME:/path ./local           # คัดลอกไฟล์ออก
docker diff NAME                       # ดูไฟล์ที่เปลี่ยนไป
```

### 💾 Volumes

```bash
docker volume create NAME
docker volume ls
docker volume inspect NAME
docker volume rm NAME
docker volume prune
docker run -v NAME:/path IMG           # named volume
docker run -v $(pwd):/app IMG          # bind mount
docker run -v /data:/data:ro IMG       # read-only
```

### 🌐 Networks

```bash
docker network create NAME
docker network ls
docker network inspect NAME
docker network connect NET CONTAINER
docker network disconnect NET CONTAINER
docker network rm NAME
docker run --network NAME IMG
```

### 🧩 Compose

```bash
docker compose up -d                   # เริ่มเบื้องหลัง
docker compose up --build              # build ใหม่แล้วเริ่ม
docker compose down                    # หยุดและลบ
docker compose down -v                 # ลบ volume ด้วย
docker compose ps                      # ดูสถานะ
docker compose logs -f SERVICE         # ดู log
docker compose exec SERVICE bash       # เข้า shell
docker compose restart SERVICE         # รีสตาร์ท
docker compose config                  # ตรวจไฟล์ YAML
```

### 🧹 Cleanup

```bash
docker system df                       # ดูพื้นที่ที่ใช้
docker system prune                    # ล้างของที่ไม่ใช้
docker system prune -a --volumes       # ⚠️ ล้างทุกอย่าง
docker container prune
docker image prune -a
docker volume prune
docker builder prune                   # ล้าง build cache
```

### 🎮 GPU

```bash
docker run --gpus all IMG
docker run --gpus 2 IMG
docker run --gpus '"device=0,1"' IMG
docker run --shm-size=8g --gpus all IMG
nvidia-smi                             # ตรวจสอบ GPU บน host
```

### 📋 Dockerfile Quick Reference

```dockerfile
FROM python:3.11-slim              # base image
ENV KEY=value                      # environment variable
ARG BUILD_VAR=default              # build-time variable
WORKDIR /app                       # working directory
COPY src dest                      # คัดลอกไฟล์
RUN command                        # รันตอน build
EXPOSE 8000                        # ประกาศพอร์ต
VOLUME /data                       # จุดเมาต์
USER appuser                       # เปลี่ยน user
HEALTHCHECK CMD curl -f localhost  # ตรวจสุขภาพ
ENTRYPOINT ["python"]              # คำสั่งหลัก
CMD ["app.py"]                     # argument เริ่มต้น
```

### 🚨 Troubleshooting

| ข้อความ Error | วิธีแก้ |
|---------------|---------|
| `port is already allocated` | `docker ps` หาตัวที่ใช้ แล้วเปลี่ยนพอร์ตหรือหยุดมัน |
| `no space left on device` | `docker system prune -a` |
| `permission denied` | เพิ่ม user เข้ากลุ่ม docker หรือใช้ `--user $(id -u)` |
| `Cannot connect to Docker daemon` | สตาร์ท Docker service / Docker Desktop |
| `exec format error` | สถาปัตยกรรมไม่ตรง (ARM vs x86) ใช้ `--platform linux/amd64` |
| `container exits immediately` | `docker logs` ดูสาเหตุ — มักเป็น CMD ที่จบทันที |
| `DataLoader worker killed` | เพิ่ม `--shm-size=8g` |
| `CUDA not available` | ตรวจ driver + `--gpus all` + เวอร์ชัน CUDA ใน image |

---

## 17. Final Challenge: Ship a Mini ML Service

### 🎯 โจทย์

สร้าง **Iris Classifier API** ที่รันด้วย Docker ทั้งหมด พร้อม Redis cache

### 📂 โครงสร้างโปรเจกต์

```
mini-ml-service/
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── requirements.txt
├── train.py
├── app.py
└── models/
    └── (model.pkl จะถูกสร้างที่นี่)
```

### 1️⃣ `requirements.txt`

```txt
fastapi==0.111.0
uvicorn[standard]==0.30.1
scikit-learn==1.5.0
numpy==1.26.4
joblib==1.4.2
pydantic==2.7.4
redis==5.0.7
```

### 2️⃣ `train.py`

```python
"""เทรนโมเดล Iris Classifier แล้วบันทึกลงไฟล์"""
import joblib
from pathlib import Path
from sklearn.datasets import load_iris
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

def main():
    X, y = load_iris(return_X_y=True)
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42, stratify=y
    )

    model = RandomForestClassifier(n_estimators=100, random_state=42)
    model.fit(X_train, y_train)

    acc = accuracy_score(y_test, model.predict(X_test))
    print(f"✅ Accuracy: {acc:.4f}")

    out_dir = Path("/models")
    out_dir.mkdir(parents=True, exist_ok=True)
    joblib.dump(model, out_dir / "model.pkl")
    print(f"💾 บันทึกโมเดลที่ {out_dir / 'model.pkl'}")

if __name__ == "__main__":
    main()
```

### 3️⃣ `app.py`

```python
"""FastAPI service สำหรับ Iris Classifier"""
import os
import json
import joblib
import numpy as np
import redis
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

MODEL_PATH = os.getenv("MODEL_PATH", "/models/model.pkl")
REDIS_URL = os.getenv("REDIS_URL", "redis://redis:6379")

CLASS_NAMES = ["setosa", "versicolor", "virginica"]

app = FastAPI(title="Iris Classifier API", version="1.0.0")

model = None
cache = None


class IrisInput(BaseModel):
    sepal_length: float = Field(..., ge=0, le=10, example=5.1)
    sepal_width: float = Field(..., ge=0, le=10, example=3.5)
    petal_length: float = Field(..., ge=0, le=10, example=1.4)
    petal_width: float = Field(..., ge=0, le=10, example=0.2)


@app.on_event("startup")
def startup():
    global model, cache
    model = joblib.load(MODEL_PATH)
    try:
        cache = redis.from_url(REDIS_URL, decode_responses=True)
        cache.ping()
    except Exception:
        cache = None  # ทำงานต่อได้แม้ไม่มี cache


@app.get("/health")
def health():
    return {
        "status": "ok",
        "model_loaded": model is not None,
        "cache_connected": cache is not None,
    }


@app.post("/predict")
def predict(data: IrisInput):
    if model is None:
        raise HTTPException(503, "Model ยังไม่พร้อมใช้งาน")

    features = [
        data.sepal_length, data.sepal_width,
        data.petal_length, data.petal_width,
    ]
    key = "iris:" + ":".join(map(str, features))

    if cache:
        hit = cache.get(key)
        if hit:
            return {**json.loads(hit), "cached": True}

    proba = model.predict_proba(np.array([features]))[0]
    idx = int(np.argmax(proba))
    result = {
        "prediction": CLASS_NAMES[idx],
        "confidence": round(float(proba[idx]), 4),
        "probabilities": {
            name: round(float(p), 4) for name, p in zip(CLASS_NAMES, proba)
        },
    }

    if cache:
        cache.setex(key, 3600, json.dumps(result))

    return {**result, "cached": False}
```

### 4️⃣ `Dockerfile`

```dockerfile
FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN useradd -m -u 1000 appuser && mkdir -p /models && chown -R appuser /app /models
USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=5s --start-period=15s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')"

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 5️⃣ `docker-compose.yml`

```yaml
services:
  trainer:
    build: .
    container_name: iris-trainer
    command: python train.py
    volumes:
      - ml-models:/models
    networks:
      - iris-net

  api:
    build: .
    container_name: iris-api
    ports:
      - "8000:8000"
    environment:
      - MODEL_PATH=/models/model.pkl
      - REDIS_URL=redis://redis:6379
    volumes:
      - ml-models:/models:ro
    depends_on:
      trainer:
        condition: service_completed_successfully
      redis:
        condition: service_healthy
    restart: unless-stopped
    networks:
      - iris-net

  redis:
    image: redis:7-alpine
    container_name: iris-redis
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
    networks:
      - iris-net

volumes:
  ml-models:

networks:
  iris-net:
    driver: bridge
```

### 6️⃣ `.dockerignore`

```gitignore
.git
__pycache__/
*.pyc
.venv/
models/*.pkl
.env
.DS_Store
README.md
```

### ▶️ วิธีรัน

```bash
# 1. build และเริ่มทั้งระบบ
docker compose up --build

# 2. ตรวจสอบสถานะ
docker compose ps

# 3. ทดสอบ health check
curl http://localhost:8000/health

# 4. ทดสอบทำนาย
curl -X POST http://localhost:8000/predict \
  -H "Content-Type: application/json" \
  -d '{"sepal_length":5.1,"sepal_width":3.5,"petal_length":1.4,"petal_width":0.2}'

# 5. เรียกซ้ำอีกครั้ง — ควรได้ "cached": true
```

**ผลลัพธ์ที่คาดหวัง:**

```json
{
  "prediction": "setosa",
  "confidence": 1.0,
  "probabilities": {"setosa": 1.0, "versicolor": 0.0, "virginica": 0.0},
  "cached": false
}
```

เปิด API docs อัตโนมัติได้ที่ **http://localhost:8000/docs**

### 🏆 Bonus Challenges

| ระดับ | โจทย์ |
|-------|-------|
| ⭐ | เพิ่ม endpoint `/model-info` แสดง feature importance |
| ⭐⭐ | แปลง Dockerfile เป็น multi-stage build แล้ววัดขนาดที่ลดลง |
| ⭐⭐ | เพิ่ม Prometheus metrics endpoint `/metrics` |
| ⭐⭐⭐ | เพิ่ม MLflow tracking ลงใน compose stack |
| ⭐⭐⭐ | สร้าง GitHub Actions ที่ build และ push image ขึ้น ghcr.io อัตโนมัติ |
| ⭐⭐⭐⭐ | Deploy ขึ้น cloud (Cloud Run / ECS / Azure Container Apps) |

### ✅ เกณฑ์ผ่าน

- [ ] `docker compose up` แล้วระบบทำงานได้ครบโดยไม่ต้องแก้อะไร
- [ ] `/health` ตอบ `status: ok` และ `model_loaded: true`
- [ ] `/predict` ทำนายได้ถูกต้อง
- [ ] เรียกซ้ำแล้วได้ `cached: true`
- [ ] `docker compose down` แล้ว `up` ใหม่ — โมเดลยังอยู่ (ไม่ต้องเทรนใหม่)
- [ ] Image ขนาดไม่เกิน 600 MB

---

## 📚 แหล่งเรียนรู้เพิ่มเติม

- [Docker Official Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Play with Docker](https://labs.play-with-docker.com/) — ลองเล่นฟรีบนเบราว์เซอร์
- [NVIDIA NGC Catalog](https://catalog.ngc.nvidia.com/) — image สำหรับ AI
- [Awesome Docker](https://github.com/veggiemonk/awesome-docker)
- [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

---

## 🎓 ก้าวต่อไปหลังจบคอร์สนี้

1. **Kubernetes** — จัดการคอนเทนเนอร์ระดับ production หลายเครื่อง
2. **CI/CD** — GitHub Actions, GitLab CI สำหรับ build/test/deploy อัตโนมัติ
3. **MLOps** — MLflow, Kubeflow, Airflow, DVC
4. **Model Serving ขั้นสูง** — Triton, KServe, Ray Serve
5. **Observability** — Prometheus, Grafana, OpenTelemetry

---

<div align="center">

**Happy Shipping! 🐳🚀**

*ถ้าเอกสารนี้มีประโยชน์ อย่าลืมกด ⭐ ให้ repo นี้*

</div>
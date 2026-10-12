# Bài 2: Tự động hóa đóng gói và phát hành Docker Image khi gắn Git Tag (v*..)

## 1. Thông tin bài tập
- **Khóa học:** DevOps
- **Buổi học:** Session 12 - CD / Release Automation
- **Bài thực hành:** Bài 2 (Tự động hóa đóng gói và phát hành Docker Image khi gắn Git Tag `v*.*.*`)
- **Thư mục bài tập:** `ss12_2`
- **GitHub Repository:** `https://github.com/HigaWateru/NguyenManhHung_DevOps_ss12_2`
- **Gói Docker Image trên GHCR:** `ghcr.io/higawateru/nguyenmanhhung_devops_ss12_2`

---

## 2. Kịch bản GitHub Actions (`.github/workflows/release.yml`)

```yaml
name: Release Docker Image

on:
  push:
    tags:
      - 'v*.*.*'
  workflow_dispatch:

# Cấp quyền đọc mã nguồn và ghi packages vào GitHub Container Registry (GHCR)
permissions:
  contents: read
  packages: write

jobs:
  release-image:
    name: Build & Release Docker Image
    runs-on: ubuntu-latest

    steps:
      # 1. Checkout mã nguồn
      - name: Checkout code
        uses: actions/checkout@v5

      # 2. Chuẩn hóa tên repository thành chữ thường (bắt buộc cho GHCR)
      - name: Set lowercase repository name for GHCR
        run: |
          echo "IMAGE_NAME=ghcr.io/${GITHUB_REPOSITORY,,}" >> $GITHUB_ENV

      # 3. Đăng nhập vào GHCR với GITHUB_TOKEN
      - name: Log in to GitHub Container Registry (GHCR)
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      # 4. Trích xuất tag & metadata từ Git Tag (v*.*.*)
      - name: Extract Docker metadata (Tags & Labels)
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=tag
            type=semver,pattern={{version}}
            type=semver,pattern=v{{version}}

      # 5. Thiết lập Docker Buildx
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # 6. Đóng gói và push Docker Image lên GHCR
      - name: Build and push Docker Image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

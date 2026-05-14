# Portfolio Project — kyuwankim/portfolio

## 프로젝트 개요
- 정적 HTML 포트폴리오 사이트
- 도메인: **https://kyuwan.kim** (사용자 안내 시 항상 이 주소 사용)
- GitHub: https://github.com/kyuwankim/portfolio.git (HTTPS, kyuwankim 계정)
- EC2 배포: ubuntu@3.35.21.82, SSH key `~/Downloads/tda-api.pem`, 경로 `/var/www/portfolio/`
- 배포 방식: `git push` → EC2에서 `git pull origin main`

## EC2 서버 구조 (중요)
- `/var/www/portfolio/` — 배포 디렉토리 (git repo)
- `/home/ubuntu/tda-data/images/` — TDA 앱에서 촬영한 원본 상품 사진 (UUID 파일명)
- `/home/ubuntu/gen_tda_viz.py` — research 시각화 이미지 생성 스크립트 (DB 연결)
- `/home/ubuntu/tda-api/` — TDA API 서버
- MySQL DB `tda_db` (user: tda / pw: tda1234) — `tda_analysis` 테이블에 상품별 이미지 경로 저장

## 절대 하지 말 것

### EC2에서 `git clean` 금지
**EC2 서버에서 절대로 `git clean -fd`를 실행하지 말 것.**
- 2026-05-12 사고: EC2에서 `git clean -fd` 실행 → git에 커밋되지 않은 research 이미지 18개 전부 삭제됨
- crop_examples/, persistence_diagram, filtration_steps 등 모두 유실
- 복구에 DB 원본 사진 + gen_tda_viz.py 재실행이 필요했음
- **EC2에 untracked 파일이 있으면 삭제하지 말고, 먼저 git에 커밋할 것**

### EC2 배포 시 충돌 해결
- `git pull`이 충돌하면 `git stash` → `git pull` → `git stash pop` 순서로 해결
- **절대로 `git checkout -- .` + `git clean -fd` 조합으로 날리지 말 것**
- untracked 파일이 있으면 먼저 `git add` + `git commit`으로 보존한 뒤 pull

## research 이미지 관리
- 모든 research 이미지는 반드시 git에 커밋해야 함 (.gitignore에 넣지 말 것)
- 새 이미지를 EC2에서 생성한 경우: `scp`로 로컬에 가져와서 git commit → push → EC2 pull
- `gen_tda_viz.py` 출력: persistence_diagram.png, persistence_barcode.png, filtration_steps.png
- crop_examples/: DB의 실제 상품 사진(kitkat, cookie, kavalan, Airpods Pro)에서 생성
  - _orig.jpg: 원본 리사이즈
  - _crop.jpg: 중앙 50% 크롭
  - _gray.jpg: 128x128 grayscale
- accuracy_comparison_v2.png, cluster_comparison.png: HTML 테이블 데이터 기반 matplotlib 차트

## 커밋 규칙
- `feat:` / `fix:` / `docs:` 등 conventional commit prefix 필수
- Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>

## 기타 주의사항
- GitHub credential: `git config credential.https://github.com.username kyuwankim` 설정됨
- .gitignore: PROGRESS.md, profile.jpg, profile_crop.jpg, profile_cutout.png (프로필 이미지만 제외)
- study.html: nav에서 제거됨, research.html 하단에 숨겨진 링크로만 접근 가능
- IntersectionObserver: Weekly Log 항목에는 `.reveal` 클래스 사용하지 않음 (해상도 이슈)

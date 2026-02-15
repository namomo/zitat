# zitat

YouTube 영상의 특정 구간을 다운로드하고, 자막을 생성·번역해서 영상에 입히는 CLI 도구.

> **zitat** — 독일어로 "인용(Zitat)"

## 설치

### 필수 도구

모두 시스템에 설치되어 있어야 합니다.

| 도구 | 설치 방법 (macOS) |
|------|-------------------|
| **yt-dlp** | `brew install yt-dlp` |
| **ffmpeg** | `brew install ffmpeg` |
| **cmake** | `brew install cmake` |
| **whisper.cpp** | 아래 참고 |

### uv 설정

```bash
uv venv
uv sync 
```

### whisper.cpp 설치

```bash
git clone https://github.com/ggerganov/whisper.cpp.git
cd whisper.cpp

# 모델 다운로드
./models/download-ggml-model.sh large-v3-turbo

# 빌드
cmake -B build
cmake --build build --config Release
```

빌드 후 `.env.example`을 복사해서 경로를 설정하세요:

```bash
cp .env.example .env
```

```bash
# .env
# 실행경로를 맞춰야 함
WHISPER_BIN=~/whisper.cpp/build/bin/whisper-cli
WHISPER_MODEL=~/whisper.cpp/models/ggml-large-v3-turbo.bin
GOOGLE_API_KEY=
```

`--whisper-bin`, `--whisper-model` CLI 옵션이나 셸 환경변수로도 지정 가능합니다.

### 자막 폰트

기본 폰트는 **BM Dohyeon**(배민 도현체)입니다. 설치되어 있지 않으면 `--font` 옵션으로 다른 폰트를 지정하세요.

- [배민 도현체 다운로드](https://www.woowahan.com/fonts)

## 사용법

```bash
python zitat.py <youtube-url> [옵션]
```

### 옵션

| 옵션 | 기본값 | 설명 |
|------|--------|------|
| `-ss`, `--start` | `0` | 시작 시간 (ffmpeg 포맷: `0:01:30`, `90` 등) |
| `-t`, `--duration` | 전체 | 길이 (초 또는 ffmpeg 포맷) |
| `-o`, `--output` | `{video_id}_ko` | 출력 파일명 (.mp4 자동 추가) |
| `--lang` | `Korean` | 번역 대상 언어 |
| `--font` | `BM Dohyeon` | 자막 폰트 |
| `--font-size` | `22` | 자막 크기 |
| `--whisper-bin` | `$WHISPER_BIN` 또는 `whisper-cli` | whisper-cli 바이너리 경로 |
| `--whisper-model` | `$WHISPER_MODEL` | whisper 모델 파일 경로 (필수) |
| `--no-review` | — | 자막 검수 단계 건너뛰기 |
| `--keep-tmp` | — | 임시 파일 보존 (디버깅용) |

### 예시

```bash
# 영상 처음 50초를 한국어 자막과 함께 추출
python zitat.py "https://youtu.be/j190mwiVlwA" -ss 0 -t 50 -o peter_test

# 1분 30초부터 2분간, 일본어로 번역
python zitat.py "https://youtu.be/j190mwiVlwA" -ss 1:30 -t 120 --lang Japanese

# 영상 전체를 다운로드해서 자막 입히기
python zitat.py --no-review "https://youtu.be/j190mwiVlwA" 

# 중간 파일 확인하면서 디버깅
python zitat.py "https://youtu.be/j190mwiVlwA" -t 30 --keep-tmp
```

## 파이프라인

```
YouTube URL
  │
  ▼
[1] yt-dlp 다운로드 (1024px 이하, --download-sections로 구간 지정)
  │
  ▼
[2] ffmpeg 오디오 추출 (16kHz mono WAV)
  │
  ▼
[3] whisper-cli 자막 생성 (SRT)
  │
  ▼
[4] claude CLI 자막 번역
  │
  ▼
[5] $EDITOR 자막 검수 (--no-review로 건너뛰기)
  │
  ▼
[6] ffmpeg 자막 입히기 (burn-in)
  │
  ▼
출력.mp4
```

## 라이선스

MIT License. 자세한 내용은 [LICENSE](LICENSE) 파일을 참고하세요.

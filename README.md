# Ren'Py Script Extraction and Translation Preparation

## Overview
이 프로젝트는 Ren'Py 게임 파일에서 스크립트와 이미지를 추출하고, `.rpyc` 파일을 디컴파일하여 텍스트 대화 스크립트를 추출하는 과정을 자동화하는 것을 목표로 합니다. 추출된 텍스트는 번역 작업을 위해 사용될 수 있습니다.

## Prerequisites
이 프로젝트를 실행하기 위해서는 다음 도구들이 필요합니다:

*   **Python 3.x**: `pip`를 포함합니다.
*   **Git**: 소스 코드 관리 및 `unrpyc` 클론을 위해 필요합니다.
*   **build-essential (Linux)**: `unrpyc` 설치를 위한 빌드 도구입니다.
*   **Node.js 및 npm (선택 사항)**: `gemini-cli` 설치를 위해 필요합니다.

## Installation

### 1. `unrpa` 설치
`unrpa`는 `.rpa` 아카이브 파일에서 내용을 추출하는 데 사용됩니다. 일반적으로 Python 패키지로 설치할 수 있습니다.

```bash
pip install unrpa
```

### 2. `unrpyc` 설치
`unrpyc`는 Ren'Py 컴파일된 스크립트 파일(`.rpyc`)을 디컴파일하여 원본 `.rpy` 파일로 되돌리는 데 사용됩니다.

```bash
# 1. unrpyc 저장소 클론
git clone https://github.com/CensoredUsername/unrpyc.git
cd unrpyc

# 2. 필요한 빌드 도구 설치 (Linux의 경우)
sudo apt install build-essential

# 3. pip, setuptools wheel twine check-wheel-contents 최신 버전으로 업데이트
python -m pip install --upgrade setuptools wheel twine check-wheel-contents

# 4. unrpyc 설치
pip install . --no-cache-dir
```

### 3. `pyinstaller` 설치 (선택 사항)
`unrpyc.py` 스크립트를 단일 실행 파일로 만들고 싶다면 `pyinstaller`를 설치합니다.

```bash
pip install pyinstaller --no-warn-script-location
```

`unrpyc.py`를 실행 파일로 변환:
```bash
pyinstaller --onefile unrpyc.py
# 생성된 실행 파일은 보통 `dist/unrpyc` 경로에 있습니다.
# 이를 `/usr/local/bin` 등으로 이동하여 전역에서 사용할 수 있도록 할 수 있습니다.

sudo mv dist/unrpyc /usr/local/bin/unrpyc
sudo chmod 755 /usr/local/bin/unrpyc
```



## Usage

### 1. RPA 아카이브 추출
`.rpa` 파일에서 게임 리소스 (이미지, 오디오 등)를 추출합니다.

```bash
# scripts.rpa 파일의 내용을 현재 디렉토리의 _workspace 폴더에 추출
unrpa -mp "./_workspace" scripts.rpa

# 다른 .rpa 파일도 동일하게 추출
unrpa -mp "./_workspace" ep1_images.rpa
```

### 2. RPYC 파일 디컴파일
컴파일된 Ren'Py 스크립트 파일(`.rpyc`)을 원본 `.rpy` 파일로 디컴파일합니다.

```bash
# 특정 .rpyc 파일을 디컴파일
unrpyc script.rpyc

# 디컴파일된 파일을 특정 디렉토리(예: 'xxx')에 저장
mkdir xxx
unrpyc script.rpyc xxx
```

### 3. 대화 스크립트 추출 (grep)
디컴파일된 `.rpy` 파일에서 대화 텍스트만 추출합니다. 이 정규식은 `character "dialogue"` 형식의 라인을 찾습니다.

```bash
# 단일 .rpy 파일에서 대화 추출
grep -E '^\s*[a-zA-Z]+\s*\".*?\"$' e1.rpy > e1.txt

# 하위 디렉토리 포함 모든 .rpy 파일에서 대화 추출
find . -name "*.rpy" -exec bash -c 'grep -E "^\\s*[a-zA-Z]+\\s*\\\"[^\\]*\\\"$" "$0" > "$(basename "$0" .rpy).txt"' {} \;
```

### 4. 추출된 스크립트 후처리 (sed)
추출된 텍스트 파일을 번역에 적합한 형태로 가공하기 위한 `sed` 명령어들입니다.

```bash
# 'add'로 시작하는 줄 제거 (예: 이미지 추가 명령)
sed -i "/^\s\+add \\\".*$"/d" *.txt

# 줄 끝의 큰따옴표 제거
sed -i 's/"$//' *.txt

# 줄 시작 부분의 캐릭터 이름과 큰따옴표 제거 (예: 'character "dialogue"' -> 'character dialogue')
sed -i 's/^\(\s*\w\+\) \"/\1 /' *.txt

# 캐릭터 약어를 전체 이름으로 변경 (예: 'mo' -> 'Eve:')
sed -i 's/\[m!t\]/Dennis/' *
sed -i 's/\[m!ct\]/Dennis/' *
sed -i 's/^\(\s*\)mo /\1Eve: /' *.txt
sed -i 's/^\(\s*\)m /\1Me: /' *.txt
sed -i 's/^\(\s*\)j /\1Jane: /' *.txt
sed -i 's/^\(\s*\)em /\1Emily: /' *.txt
sed -i 's/^\(\s*\)k /\1Kiara: /' *.txt
sed -i 's/^\(\s*\)n /\1Narr: /' *.txt
sed -i 's/^\(\s*\)nao /\1Naomi: /' *.txt
sed -i 's/^\(\s*\)v /\1Violet: /' *.txt
sed -i 's/^\(\s*\)b /\1Belle: /' *.txt
sed -i 's/^\(\s*\)s /\1Stacy: /' *.txt
sed -i 's/^\(\s*tut\) /\1:\t/' *.txt
sed -i 's/^\(\s*hint\) /\1:\t/' *.txt
sed -i 's/^\(\s*noname\) /\1:\t/' *.txt
sed -i 's/^\(\s*svi\) /\1:\t/' *.txt
sed -i 's/^\(\s*voice\) /\1:\t/' *.txt


# 캐릭터 이름 뒤에 탭 추가 (정렬을 위해)
sed -i 's/^\(\s*\w\+:\) /\1\t/' *.txt

# Ren'Py 서식 태그 제거 (예: {i}, {/i})
sed -i 's/{\/\?i}//g' *.txt

# 줄바꿈 문자 '\n'을 공백으로 대체
sed -i 's/\\n/ /g' *.txt

# 이스케이프된 큰따옴표 '\"'를 일반 큰따옴표 '"'로 대체
sed -i 's/\\"/"/g' *.txt

# html  변환
sed -i 's/\(\s*\)\(\w\+:\)/\1<div class="line"><span class="name">\2<\/span><div>/' e01.html
sed -i 's/$/<\/div><\/div>/' e01.html
sed -i 's/^\(\s\+\)<div class="line">/<div class="line">\1/' e01.html
perl -i -pe '1 while s/( *?) {4}/$1<span class="indent"><\/span>/' e01.html
sed -i '1i\
<!DOCTYPE html>\
<html>\
<meta charset="utf-8">\
<head>\
\t<link rel="stylesheet" href="style.css">\
</head>\
<body>' e01.html
printf '</body>\n</html>\n' >> e01.html

find . -name "*.txt" -exec echo $0 {} \;

find . -name "*.txt" | while IFS= read -r file; do echo "$0" "$file"; done


# copy *.txt *.html
for file in *.txt; do
   cp $file "$(basename "$file" .txt).html"
done

# html 변환 일괄
sed -i 's/\(\s*\)\(\w\+:\)/\1<div class="line"><span class="name">\2<\/span><div>/' *.html
sed -i 's/$/<\/div><\/div>/' *.html
sed -i 's/^\(\s\+\)<div class="line">/<div class="line">\1/' *.html
perl -i -pe '1 while s/( *?) {4}/$1<span class="indent"><\/span>/' *.html
sed -i '1i\
<!DOCTYPE html>\
<html>\
<meta charset="utf-8">\
<head>\
\t<link rel="stylesheet" href="style.css">\
</head>\
<body>' *.html
printf '</body>\n</html>\n' >> *.html
```

## License
이 프로젝트는 `LICENSE` 파일에 명시된 라이선스를 따릅니다.

```

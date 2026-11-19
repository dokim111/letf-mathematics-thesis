# 유계 일수익률에서 레버리지 로그감쇠의 고차 근사오차와 극값 전이

이 논문은 **일간 수익률이 유계일 때 레버리지 로그감쇠의 근사오차를 얼마나 좁게 경계할 수 있는지, 오차의 극값 위치가 배율과 근사 차수에 따라 어떻게 바뀌는지**를 연구합니다. 이 저장소에는 해당 수식과 상수를 검산하는 코드, 실행 환경, 이론 계산 결과와 실제 종목의 수치 예시를 재생성하는 절차를 제공합니다.

## 연구 질문과 계산 대상

하루의 기초자산 수익률이 $r$, 목표 배율이 $\beta$일 때 로그감쇠는 다음과 같습니다. 정의역은 $1+r>0$, $1+\beta r>0$인 구간입니다.

$$
g_\beta(r)=\beta\log(1+r)-\log(1+\beta r).
$$

$n$차 테일러 다항식 $T_n$의 나머지를 $r^{n+1}\rho_{n,\beta}(r)$로 정규화하여 다음을 다룹니다.

- 정규화 나머지항의 부호, 단조성·볼록성 및 쌍대성.
- 홀수 차수에서, $T_n$을 고정하고 $r^{n+1}$의 상수배로 오차를 경계하는 계열의 최적 상수와 극값 위치.
- 배율에 따른 극값 전이와 차수별 임계값 $b_m$의 단조성·양측 경계·점근식.
- 이차·삼차 근사의 누적 오차와 경계 구간을 실제 수익률 경로에서 계산하는 수치 예시.

수치 검산은 구현과 예시를 확인하는 절차이며 해석적 증명을 대신하지 않습니다. **이론 검산은 시장 원자료 없이 실행할 수 있습니다.** 종목별 수치 예시에는 별도로 확보한 KRX 자료가 필요합니다.

## 실행 환경

Python 3.12 계열에서 사용한 패키지 버전은 `requirements.txt`에 고정했습니다. 가상환경을 준비하고 저장소 루트에서 실행합니다. 아래의 모든 상대경로는 저장소 루트를 기준으로 합니다.

```bash
python3 -m pip install -r requirements.txt
```

모든 계산은 결정론적이며 난수와 시드를 사용하지 않습니다. 플랫폼과 수치 라이브러리에 따라 마지막 몇 자리는 달라질 수 있습니다. 그림 생성에는 Noto Sans CJK KR, NanumGothic 등 한글 글꼴이 필요합니다. 이론 검산에는 글꼴이 필요하지 않습니다.

## 이론 검산 실행

다음 순서로 실행합니다. 마지막 명령은 바로 앞에서 생성한 `order_thresholds.csv`를 읽습니다.

```bash
python3 theory_checks.py
python3 remainder_checks.py
python3 general_theory_checks.py
python3 order_threshold_checks.py
python3 order_rate_checks.py
```

| 파일 | 계산 내용 | 정밀도·출력 |
|---|---|---|
| `theory_checks.py` | 저차 경계 상수, 음의 배율의 최소점과 초기 전이값 | mpmath 40자리, 표준 출력 |
| `remainder_checks.py` | 적분 나머지항과 저차식의 기호·수치 검산 | mpmath 30자리, SymPy, 표준 출력 |
| `general_theory_checks.py` | 일반 차수의 쌍대성·미분식·극값 구조 | mpmath 40자리, JSON 형식 표준 출력 |
| `order_threshold_checks.py` | 차수별 임계값과 상계, 치환 적분 항등식 | SciPy 적분·Brent 근구하기; 선택 차수를 mpmath 60자리로 대조. `order_thresholds.csv`, `order_threshold_checks_output.json` |
| `order_rate_checks.py` | 임계값 수열의 양측 경계와 점근식 검산 | mpmath 60자리. `order_rate_checks.csv`, `order_rate_checks_output.json` |

`order_thresholds.csv`는 `m=0,...,80`의 `b_m`, 해석적 상계 `b_bar_m`, 잔차 등을 담습니다. `order_rate_checks.csv`는 차수에 따른 수렴률의 계산 결과입니다. 세 JSON은 시장자료나 실행 경로를 포함하지 않는 이론 검산의 참고 출력입니다. `general_theory_checks.py`는 JSON을 표준 출력으로 내보내므로 새 결과는 터미널에서 확인하거나 별도 파일로 저장하십시오.

## 실제 종목의 수치 예시

삼성전자와 SK하이닉스의 일별 수익률에 대한 누적 로그감쇠, 근사값, 경계와 좌표 비교를 계산합니다. 입력 파일을 다음 위치에 준비합니다.

```text
data/krx_stock_daily_long_20000104_20261113.csv
```

출처는 [KRX 정보데이터시스템](https://data.krx.co.kr/)의 개별종목 시세입니다. 입력 열·기준가 처리·기간과 표본 수는 [DATA_SOURCES.md](DATA_SOURCES.md)에 설명합니다. 원자료는 이 저장소에 포함하지 않습니다.

```bash
mkdir -p data
python3 math_figs_tables.py data/krx_stock_daily_long_20000104_20261113.csv
```

계산 결과는 표준 출력과 `math_tables.json`, `fig/fig_h.png`, `fig/fig_bounds.png`에 생성됩니다. 상수 계산은 mpmath 30자리입니다. 표·그림은 제목을 기준으로 다음과 같이 대응합니다.

| 논문의 표·그림 내용 | 생성 코드·출력 |
|---|---|
| 차수에 따른 전이 임계값과 해석적 상계 | `order_threshold_checks.py` → `order_thresholds.csv` |
| $c=0.30$에서의 상수 | `theory_checks.py`, `remainder_checks.py`; `math_figs_tables.py`의 상수 출력 |
| 누적 로그감쇠와 경계 | `math_figs_tables.py` → 표준 출력, `math_tables.json`의 `DATA` |
| 단순·로그수익률 좌표와 삼차 경계 비교 | `math_figs_tables.py` → 표준 출력, `math_tables.json`의 `COORD` |
| 정규화 감쇠 함수와 연도별 누적 경계 그림 | `math_figs_tables.py` → `fig/` |

## 파일과 인용

Python 코드 6개, `requirements.txt`, 이론 상수 CSV 2개, 검산 JSON 3개와 재현 안내 문서를 제공합니다. 전체 논문, KRX 원자료, 개인별 제출 서류 및 비공개 검토 기록은 포함하지 않습니다.

저장소 버전은 `VERSION`, 인용용 메타데이터는 `CITATION.cff`, 변경 이력은 [CHANGELOG.md](CHANGELOG.md)에 기록합니다. 이 자료를 인용할 때에는 저장소 버전과 사용한 커밋 SHA를 함께 특정하십시오.

저장소: [dokim111/letf-mathematics-thesis](https://github.com/dokim111/letf-mathematics-thesis)

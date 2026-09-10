# pykiwoom

키움증권 Open API+를 Python에서 사용할 수 있도록 감싼 라이브러리입니다. 로그인, 종목 정보 조회, TR 요청, 실시간 데이터, 조건검색, 주문을 지원하며 `KiwoomManager`를 사용하면 Open API+ 연동을 별도 프로세스로 분리할 수 있습니다.

> 이 저장소는 Windows용 Open API+ OCX를 사용하는 라이브러리입니다. 키움 REST API용 라이브러리가 아닙니다.

## 시작 전 확인

실행 전에 다음 환경이 필요합니다.

- Windows
- 키움증권 계좌와 HTS ID
- [키움 Open API+ 사용 등록 및 모듈 설치](https://www.kiwoom.com/m/customer/download/VOpenApiInfoView)
- 32비트 Python 환경

Open API+는 OCX 기반이며, 프로젝트의 [개발환경 안내](https://wikidocs.net/156336)는 32비트 Python 사용을 안내합니다. 현재 Python의 비트 수는 다음 명령으로 확인할 수 있습니다.

```powershell
python -c "import struct; print(struct.calcsize('P') * 8)"
```

`32`가 출력되는 환경에서 설치를 진행하세요.

`block_request()`는 TR 명세를 기본 설치 경로인 `C:\OpenAPI\data`에서 읽습니다. Open API+ 모듈은 기본 경로에 설치하는 것을 권장합니다.

## 설치

PyPI에 배포된 원 프로젝트의 패키지는 다음 명령으로 설치합니다.

```powershell
python -m pip install pykiwoom
```

이 저장소의 `master` 버전을 직접 설치하려면 다음 명령을 사용합니다.

```powershell
python -m pip install "git+https://github.com/uckdo/pykiwoom.git"
```

소스를 수정하면서 사용하려면 저장소를 복제한 뒤 편집 가능한 모드로 설치합니다.

```powershell
git clone https://github.com/uckdo/pykiwoom.git
cd pykiwoom
python -m pip install -e .
```

`setup.py`에는 `pandas`, `PyQt5`, `pywin32`가 의존성으로 등록되어 있습니다. 지원 Python 버전과 의존성 버전은 고정되어 있지 않으므로, 32비트 패키지 설치가 실패하면 [개발환경 안내의 Python 3.8 구성](https://wikidocs.net/156336)을 참고하세요.

## 빠른 시작

로그인 창을 열고 연결 상태와 종목명을 확인하는 예제입니다.

```python
from pykiwoom.kiwoom import Kiwoom

kiwoom = Kiwoom()
kiwoom.CommConnect(block=True)

print(kiwoom.GetConnectState())
print(kiwoom.GetMasterCodeName("005930"))
```

`CommConnect(block=True)`는 로그인 이벤트가 끝날 때까지 기다립니다.

## TR 조회

`block_request()`는 TR 응답을 기다린 뒤 `pandas.DataFrame`으로 반환합니다.

```python
from pykiwoom.kiwoom import Kiwoom

kiwoom = Kiwoom()
kiwoom.CommConnect(block=True)

df = kiwoom.block_request(
    "opt10001",
    종목코드="005930",
    output="주식기본정보",
    next=0,
)

print(df)
```

TR 코드, 입력값, 출력 레코드 이름은 KOA Studio 또는 키움 Open API+ 개발가이드에서 확인하세요.

## 별도 프로세스로 사용하기

GUI나 다른 작업과 Open API+ 이벤트 처리를 분리하려면 `KiwoomManager`를 사용할 수 있습니다. Windows의 멀티프로세싱 동작을 위해 실행 코드는 반드시 `if __name__ == "__main__":` 안에 두세요.

```python
from pykiwoom import KiwoomManager


if __name__ == "__main__":
    manager = KiwoomManager()
    manager.put_method(("GetMasterCodeName", "005930"))
    name = manager.get_method()
    print(name)
```

TR 요청 결과는 데이터와 연속 조회 여부를 함께 반환합니다.

```python
from pykiwoom import KiwoomManager


if __name__ == "__main__":
    manager = KiwoomManager()
    request = {
        "rqname": "opt10001",
        "trcode": "opt10001",
        "next": "0",
        "screen": "1000",
        "input": {"종목코드": "005930"},
        "output": ["종목코드", "종목명", "PER", "PBR"],
    }

    manager.put_tr(request)
    data, remained = manager.get_tr()
    print(data)
    print("연속 조회 가능:", remained)
```

연속 조회가 필요하면 `remained`를 확인한 뒤 다음 요청의 `next` 값을 `"2"`로 바꾸어 다시 요청합니다.

## 예제 모음

- [`example/`](./example): 로그인, 계좌 정보, 종목 정보, 조건검색, 주문
- [`example_tr/`](./example_tr): 주요 TR의 직접 조회
- [`example_manager/`](./example_manager): 별도 프로세스, TR 연속 조회, 실시간 데이터

## 사용 시 주의사항

- 주문 예제는 실제 계좌에 영향을 줄 수 있습니다. 실환경 적용 전에 모의투자에서 충분히 확인하세요.
- Open API+의 TR 호출 제한과 운영 정책은 [키움증권 공식 안내](https://www.kiwoom.com/m/customer/download/VOpenApiInfoView)를 기준으로 확인하세요.
- 로그인 창과 OCX가 필요한 구조이므로 Linux, macOS, 일반적인 헤드리스 서버에서는 실행할 수 없습니다.
- 이 저장소에는 계좌 정보나 로그인 정보를 커밋하지 마세요.

## 문서

- [퀀트투자를 위한 키움증권 API](https://wikidocs.net/book/1173)
- [키움 Open API+ 공식 안내](https://www.kiwoom.com/m/customer/download/VOpenApiInfoView)
- [키움 Open API+ 개발가이드](https://download.kiwoom.com/web/openapi/kiwoom_openapi_plus_devguide_ver_1.7.pdf)

이 저장소는 [`sharebook-kr/pykiwoom`](https://github.com/sharebook-kr/pykiwoom)을 기반으로 합니다.

## 라이선스

라이선스 조건은 [`LICENSE`](./LICENSE) 파일을 확인하세요.

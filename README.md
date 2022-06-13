# gachon-fintech-wage
가천대학교 금융수학과 2022-1학기 핀테크 수업 알바비를 부탁해 팀 깃허브


# 월근무시간과 월급액 산출
branch 데이터를 통해 월 근무시간과 월급액 산출은 저희 프로젝트 4단계중 2단계에 해당합니다.

wage_calculate_with_adu.ipynb
최종 결과물과 관련된 파일입니다.
해당 파일 IN[71] 부분은 데이터프레임 파일 안의 값이 INT가 아니기 때문에 밑에 SUM 함수를 이용하고자 데이터프레임을 재생성하는 코드입니다.
IN[72]에서 주별 근무시간을 구하고 for문을 통해 주휴수당을 계산합니다.

wage_time data(ex).ipynb
아두이노 결과값을 받기 전 데이터 프레임의 예시를 만드는 파일입니다.

wage_calculate.ipynb
예시로 만들어진 데이터 프레임을 통해 월급액을 계산하는 파일입니다.



# 예정 지급 월급액과 지갑 잔액 비교
branch 예정 지급 월급액과 지갑 잔액 비교는 저희 프로젝트 4단계중 2단계에 해당합니다.


coinone_api_krw.ipynb 
코인원 api를 통해 지갑의 잔액을 읽어오는 파일입니다.

ACCESS_TOKEN = '52b36f87-c585-4786-85da-7f4af2208585'
SECRET_KEY = bytes('d2097823-9f5d-4851-812c-eea7f866012a', 'utf-8') 

계좌를 발급 받을 때 시크릿 키와 엑세스 토큰을 발급받아 개인 지갑의 api를 가져올 수 있습니다. 위의 키와 토큰은 저희 계좌의 것으로 계좌에는 현재 3원이 있습니다.
IN[89]는 저희가 가져 온 API 중 KRW에 해당하는 금액만을 FLOAT로 나타낸 것입니다.


transfer_possibility.ipynb
송금 가능성을 계산한 파일입니다.
앞의 내용은 wage_calculate_with_adu.ipynb과 coinone_api_krw.ipynb의 코드를 사용합니다.
IN[8]에서 지갑의 KRW 잔액과 계산된 예정 지급 월급액을 비교해 IF문을 통해 possibility을 따집니다.

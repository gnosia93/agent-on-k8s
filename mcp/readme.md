
호텔 예약 시스템의 리소스, 도구, 프롬프트를 모두 포함한 FastMCP 라이브러리 기반의 서버 예제입니다.
이 코드를 hotel_mcp.py로 저장하고 실행해 보세요.

```python
from mcp.server.fastmcp import FastMCP
# MCP 서버 생성
mcp = FastMCP("HotelReservationServer")
# --- [가상 데이터 데이터베이스] ---
HOTELS = [
    {"id": "H1", "name": "그랜드 하얏트 서울", "city": "서울", "price": 300000},
    {"id": "H2", "name": "파크 하얏트 서울", "city": "서울", "price": 450000},
    {"id": "H3", "name": "라한셀렉트 경주", "city": "경주", "price": 200000},
]
MY_BOOKINGS = []
# --- 1. 리소스 (Resources): "정보 보기" ---
@mcp.resource("hotels://list")
def list_all_hotels() -> str:
    """등록된 모든 호텔 목록을 보여줍니다."""
    return "\n".join([f"[{h['id']}] {h['name']} - {h['city']} ({h['price']}원)" for h in HOTELS])
@mcp.resource("user://my-bookings")
def get_user_bookings() -> str:
    """나의 현재 예약 내역을 보여줍니다."""
    if not MY_BOOKINGS:
        return "예약 내역이 없습니다."
    return "\n".join(MY_BOOKINGS)
# --- 2. 도구 (Tools): "액션 실행" ---
@mcp.tool()
def search_hotels(city: str) -> list:
    """특정 도시의 호텔을 검색합니다."""
    return [h for h in HOTELS if h["city"] == city]
@mcp.tool()
def book_hotel(hotel_id: str, guest_name: str) -> str:
    """호텔을 예약합니다."""
    hotel = next((h for h in HOTELS if h["id"] == hotel_id), None)
    if hotel:
        booking_info = f"예약 성공: {guest_name}님 - {hotel['name']}"
        MY_BOOKINGS.append(booking_info)
        return booking_info
    return "해당 호텔 ID를 찾을 수 없습니다."
# --- 3. 프롬프트 (Prompts): "메뉴판" ---
@mcp.prompt()
def plan_vacation(city: str) -> str:
    """휴가 계획을 도와주는 전문가용 프롬프트 템플릿입니다."""
    return f"""
당신은 여행 전문가입니다. 
1. 먼저 'hotels://list'(리소스)를 참고해서 {city} 지역에 어떤 호텔이 있는지 확인해줘.
2. 호텔 가격과 특징을 분석해서 사용자에게 추천해줘.
3. 사용자가 결정하면 'book_hotel'(도구)을 써서 예약을 도와줘.
{city} 여행을 아주 멋지게 계획해줘!
"""
if __name__ == "__main__":
    mcp.run()
```
💡 이 예제의 포인트!
* hotels://list (리소스): AI는 대화를 시작하자마자 어떤 호텔들이 있는지 **데이터(전체 목록)**를 미리 훑어볼 수 있습니다.
* search_hotels (도구): 사용자가 "서울에 있는 호텔만 골라줘"라고 하면 AI가 **입력값("서울")**을 넣어서 실행합니다.
* book_hotel (도구): 실제로 **데이터를 변경(MY_BOOKINGS에 추가)**하는 중요한 행위입니다.
* plan_vacation (프롬프트): 사용자가 "휴가 계획 짜줘"라고 하면 AI에게 **"너는 여행 전문가야 + 이 리소스를 읽고 + 이 도구를 써서 예약까지 해"**라는 복잡한 지침을 한 번에 전달합니다.

🚀 테스트 방법 (Claude Desktop)
claude_desktop_config.json에 위 파일 경로를 등록하고 재시작하면, Claude가 다음과 같이 행동합니다.

질문: "나 휴가 좀 보내줘 (프롬프트 선택)"
AI: "어느 도시로 가고 싶으신가요?" (프롬프트 지침에 따라 질문)
질문: "서울!"
AI: hotels://list 리소스를 읽어보고 search_hotels 도구를 써서 결과를 보여준 뒤 예약을 제안합니다.
이 코드를 직접 실행해 보시면서 AI가 어떻게 정보를 읽고 도구를 쓰는지 관찰해 보시면 완벽합니다! 😄🦾


---

Claude Desktop에 MCP 서버를 등록하는 상세 방법을 단계별로 안내해 드립니다.

1단계: hotel_mcp.py의 전체 경로 확인
터미널에서 hotel_mcp.py 파일이 있는 폴더로 이동한 뒤, 아래 명령어를 입력하여 전체 경로를 복사해 두세요.

bash
pwd
만약 결과가 /Users/automake/projects/mcp-test라면, 전체 파일 경로는 /Users/automake/projects/mcp-test/hotel_mcp.py가 됩니다.

2단계: 설정 파일(claude_desktop_config.json) 열기
Mac 기준으로 설정 파일은 아래 경로에 숨겨져 있습니다. 터미널에서 다음 명령어를 입력하면 텍스트 편집기(TextEdit)로 바로 열립니다.

bash
open -e ~/Library/Application\ Support/Claude/claude_desktop_config.json
3단계: JSON 데이터 추가하기
파일이 열리면 아래 내용을 붙여넣으세요. 주의: YOUR_FULL_PATH 부분은 1단계에서 복사한 실제 전체 경로로 바꿔야 합니다.

json
{
  "mcpServers": {
    "hotel-reservation": {
      "command": "python3",
      "args": [
        "/YOUR_FULL_PATH/hotel_mcp.py"
      ]
    }
  }
}
만약 파일에 이미 다른 서버 설정이 있다면, "hotel-reservation": { ... } 부분만 쉼표(,)로 구분해서 추가하면 됩니다.
저장(Command + S) 후 파일을 닫습니다.
4단계: Claude Desktop 재시작
파일 수정 사항을 적용하려면 앱을 완전히 종료해야 합니다.

상단 메뉴 바의 Claude 아이콘을 클릭하고 Quit Claude를 누릅니다.
Launchpad나 Spotlight에서 Claude 앱을 다시 실행합니다.
5단계: 연결 확인 (가장 중요한 부분!)
Claude 앱이 켜지면 채팅창 하단 입력창 주위를 살펴보세요.

🔧 아이콘: 입력창 왼쪽에 망치/렌치 모양의 아이콘이 생겼는지 확인합니다.
도구 목록: 아이콘을 클릭했을 때 search_hotels, book_hotel이 보이면 성공입니다.
리소스/프롬프트: hotels://list 리소스와 plan_vacation 프롬프트도 메뉴에서 선택 가능한지 확인해 보세요.
💡 팁: 만약 아이콘이 안 보인다면?
경로 오타: 파이썬 실행 경로(python3)가 환경 변수에 설정되어 있는지, 파일 경로에 오타가 없는지 다시 확인해 보세요.
라이브러리 미설치: pip install "mcp[cli]"를 수행했는지 확인하세요.
로그 확인: 문제가 있다면 Claude 앱의 Settings > Developer > View Logs에서 에러 메시지를 확인할 수 있습니다.

----

리모트(Remote) MCP 서버는 로컬 파일이 아니라 **인터넷 주소(URL)**를 통해 연결됩니다. 주로 SSE(Server-Sent Events) 라는 방식을 사용하는데, claude_desktop_config.json 설정 방법이 로컬과는 약간 다릅니다.

🌐 리모트 MCP 서버 등록 방법
리모트 서버는 직접 실행하는 파일이 없으므로, **npx**라는 도구를 빌려와서 해당 주소로 연결을 시도하게 만듭니다.

1. 설정 파일(claude_desktop_config.json) 수정
아래와 같이 mcpServers 안에 내용을 추가합니다.

json
{
  "mcpServers": {
    "my-remote-server": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/inspector",
        "sse",
        "https://your-remote-server.com/sse"
      ]
    }
  }
}
https://your-remote-server.com/sse: 이 부분이 실제 리모트 서버의 주소입니다.
@modelcontextprotocol/inspector: 이 도구가 리모트 서버와 Claude 사이에서 통신을 중계해줍니다.
🔍 왜 방식이 다른가요?
구분	로컬 (Local)	리모트 (Remote)
통신 방식	Stdio (표준 입출력)	SSE / HTTP (네트워크 통신)
명령어	python3 server.py	npx ... sse URL
대상	내 컴퓨터의 파일	인터넷 고정 IP/도메인
🛠️ 실제 리모트 서버 예시 (테스트용)
만약 인터넷에 공개된 샘플 리모트 서버가 있다면 이렇게 넣을 수 있습니다.

json
"remote-test": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/inspector", "sse", "https://api.example.com/mcp/sse"]
}
💡 주의사항
Node.js 설치: npx를 사용하기 때문에 내 컴퓨터에 Node.js가 설치되어 있어야 합니다.
보안(CORS/Auth): 리모트 서버는 아무나 접속하면 안 되므로 보통 API Key나 인증 절차가 필요할 수 있습니다. 서버 사양에 따라 인증 헤더(Header) 등을 추가해야 할 수도 있습니다.
네트워크 상태: 로컬 서버와 달리 인터넷 연결이 끊기면 도구가 작동하지 않습니다.
요약하자면: 리모트 서버는 npx를 이용해 인터넷 주소(URL)를 적어주는 방식으로 등록하면 됩니다

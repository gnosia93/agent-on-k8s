
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

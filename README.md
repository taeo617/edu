import React, { useState, useEffect, useRef } from 'react';

// --- Global Config ---
const apiKey = ""; 
const appId = typeof __app_id !== 'undefined' ? __app_id : 'opic-ih-master';

// --- 1. DATA Constants ---

const INTRO_LIMIT = 30; 
const MOCK_LIMIT_SEC = 40; 

const comboFlowData = [
  { num: 1, label: "자기소개", color: "bg-slate-900", border: "border-slate-900", textColor: "text-white" },
  { num: 2, label: "유형 1 (장소/사물 묘사)", color: "bg-blue-100", border: "border-blue-200", textColor: "text-blue-900" },
  { num: 3, label: "유형 2 (활동/루틴)", color: "bg-blue-100", border: "border-blue-200", textColor: "text-blue-900" },
  { num: 4, label: "유형 3 (최근 경험)", color: "bg-blue-100", border: "border-blue-200", textColor: "text-blue-900" },
  { num: 5, label: "유형 1 (장소/사물 묘사)", color: "bg-blue-50/50", border: "border-blue-100", textColor: "text-blue-800" },
  { num: 6, label: "유형 3 (최근 경험)", color: "bg-blue-50/50", border: "border-blue-100", textColor: "text-blue-800" },
  { num: 7, label: "유형 4 (인상적인 경험)", color: "bg-blue-50/50", border: "border-blue-100", textColor: "text-blue-800" },
  { num: 8, label: "유형 1 (장소/사물 묘사)", color: "bg-blue-100", border: "border-blue-200", textColor: "text-blue-900" },
  { num: 9, label: "유형 3 (최근 경험)", color: "bg-blue-100", border: "border-blue-200", textColor: "text-blue-900" },
  { num: 10, label: "유형 4 (인상적인 경험)", color: "bg-blue-100", border: "border-blue-200", textColor: "text-blue-900" },
  { num: 11, label: "유형 6 (RP - 정보 요청)", color: "bg-blue-50/50", border: "border-blue-100", textColor: "text-blue-800" },
  { num: 12, label: "유형 7 (RP - 문제 해결)", color: "bg-blue-50/50", border: "border-blue-100", textColor: "text-blue-800" },
  { num: 13, label: "유형 8 (과거 해결 경험)", color: "bg-blue-50/50", border: "border-blue-100", textColor: "text-blue-800" },
  { num: 14, label: "유형 9 (비교 및 대조)", color: "bg-rose-50", border: "border-rose-100", textColor: "text-rose-900" },
  { num: 15, label: "유형 10 (사회적 이슈)", color: "bg-rose-50", border: "border-rose-100", textColor: "text-rose-900" },
];

const surveyCategories = [
  { title: "여가 활동 (두 개 이상 선택)", items: ["영화보기", "클럽 가기", "공연보기", "콘서트보기", "박물관가기", "공원가기", "캠핑하기", "해변가기", "스포츠 관람"], strategy: ["영화보기", "공연보기", "콘서트보기", "공원가기", "캠핑하기"] },
  { title: "취미나 관심사 (한 개 이상 선택)", items: ["음악 감상하기", "악기 연주하기", "혼자 노래부르기", "춤추기", "요리하기"], strategy: ["음악 감상하기"] },
  { title: "운동 즐기기 (한 개 이상 선택)", items: ["농구", "축구", "골프", "수영", "자전거", "조깅", "걷기", "요가", "하이킹", "운동을 전혀 하지 않음"], strategy: ["조깅", "걷기", "운동을 전혀 하지 않음"] },
  { title: "휴가나 출장 경험 (한 개 이상 선택)", items: ["집에서 보내는 휴가", "국내 여행", "해외 여행"], strategy: ["집에서 보내는 휴가", "국내 여행", "해외 여행"] }
];

const masterData = {
  "공원 가기": { expected: ["Tell me about your favorite park that you often visit. What does it look like and what kind of people visit there?", "What do you usually do when you go to the park? Please describe your routine from start to finish in detail.", "Tell me about a memorable experience you had at a park recently. Who were you with and what exactly happened?"], scripts: [{ type: "묘사 (Description)", text: "Actually, there's a park called Blue Park near my place. It's the closest one, taking less than 10 minutes on foot. The scenery is absolutely **picturesque**, and I love the peaceful vibe there. Many families come here on weekends, which creates a very lively yet relaxing atmosphere.", trans: "사실 집 근처에 블루 파크라는 공원이 있어요. 걸어서 10분도 안 걸리죠. 풍경이 정말 그림 같고 평화로운 분위기를 좋아해요. 주말에는 많은 가족들이 찾아와서 활기차면서도 편안한 분위기를 자아냅니다." }, { type: "루틴 (Routine)", text: "Usually, I visit the park to get some fresh air. I start with simple stretching, and then walk three laps around the park while listening to music. It's the best way to **unwind** after a long day of work.", trans: "보통 바람을 쐬러 공원에 가요. 스트레칭으로 시작해서 음악을 들으며 공원을 세 바퀴 정도 돌죠. 긴 하루 일과를 마치고 긴장을 풀기에 가장 좋은 방법이에요." }, { type: "경험 (Experience)", text: "Oh, I have a really **embarrassing** story. One day, I ran into my ex-boyfriend at the park! He was with a new girl. I was wearing baggy sweatsuits with no makeup on. I just pretended not to notice him and ran away quickly.", trans: "정말 당황스러운 이야기가 있어요. 어느 날 공원에서 전남친을 마주친 거예요! 그는 새 여친과 있었죠. 무릎 나온 트레이닝복에 노메이크업 상태였어요. 그냥 못 본 척하고 빨리 도망쳤어요." }] },
  "조깅/걷기": { expected: ["Where do you usually go for a walk or jogging? Please describe the place and tell me why you like going there.", "Tell me about your routine for walking or jogging. What do you prepare before you head out?", "Have you ever had any unexpected incident while walking? Please explain the situation."], scripts: [{ type: "묘사 (Description)", text: "I often go jogging at the riverside park near my apartment. I first became interested in it to build up my **stamina**. Now, it helps me clear my mind whenever I'm feeling stressed.", trans: "아파트 근처 강변 공원에서 자주 조깅해요. 체력을 기르려고 시작했는데, 지금은 스트레스 받을 때 머리를 식히는 데 큰 도움이 돼요." }, { type: "루틴 (Routine)", text: "My routine is simple. I put on my running shoes, grab my water bottle, and do warm-up exercises first. Then I do **interval training** for 30 minutes. It makes me feel energized.", trans: "제 루틴은 간단해요. 러닝화를 신고 물통을 챙긴 뒤 준비 운동을 먼저 합니다. 그러고 나서 30분 동안 인터벌 트레이닝을 하죠. 정말 활력이 넘치는 기분이 들어요." }, { type: "경험 (Experience)", text: "Once, I **sprained my ankle** while jogging. I was listening to music so loudly that I didn't see a small rock on the path. I had to have a cast for 3 weeks and stay at home.", trans: "한번은 조깅하다 발목을 삐었어요. 음악을 너무 크게 듣느라 길에 있는 작은 돌을 못 봤거든요. 3주 동안 깁스를 하고 집에만 있어야 했죠." }] },
  "음악 감상": { expected: ["What kind of music do you like these days? Who is your favorite singer and why?", "When and where do you usually listen to music? Do you listen to it more often at home or while commuting?", "How has your taste in music changed from the past to now?"], scripts: [{ type: "묘사 (Description)", text: "I'm a huge fan of IU. She has a **crystal-clear voice** and tons of hit songs. Her music is simply out of this world, and it always tops the charts immediately after release.", trans: "전 아이유의 광팬이에요. 그녀는 맑고 깨끗한 목소리와 수많은 히트곡을 가지고 있죠. 그녀의 음악은 정말 최고예요. 나오자마자 차트를 휩쓸죠." }, { type: "루틴 (Routine)", text: "I usually listen to music while commuting. It helps me concentrate and get in the zone. I always use my wireless earbuds to fully **immerse** myself in the melody.", trans: "보통 이동할 때 음악을 들어요. 집중하고 몰입하는 데 도움이 되거든요. 멜로디에 완전히 몰입하기 위해 항상 무선 이어폰을 사용합니다." }, { type: "경험 (Experience)", text: "I went to a K-pop concert last year and was completely **blown away** by the performance. The atmosphere was so energetic and everyone was singing along. It was an unforgettable night.", trans: "작년에 K-pop 콘서트에 갔었는데 공연에 완전히 압도당했어요. 분위기가 정말 활기찼고 모든 사람이 떼창을 했죠. 잊지 못할 밤이었어요." }] },
  "여행 (국내/해외)": { expected: ["Tell me about a city you visited recently. What were your first impressions?", "What do you do to prepare for a trip? Please explain your preparation process.", "Talk about an unexpected problem during a trip. How did you handle that situation?"], scripts: [{ type: "묘사 (Description)", text: "I love visiting the east coast. The ocean view is absolutely **stunning** and the air is so fresh. It's the best place to recharge after a long, stressful week.", trans: "전 동해안에 가는 걸 좋아해요. 바다 풍경이 정말 아름답고 공기도 아주 신선하죠. 스트레스 가득한 한 주를 보내고 재충전하기에 가장 좋은 곳이에요." }, { type: "루틴 (Routine)", text: "I'm a total **heavy packer**. I always check the weather and pack everything 'just in case'. Before leaving, I double-check my itinerary to ensure a smooth trip without any issues.", trans: "전 정말 짐을 많이 싸는 스타일이에요. 날씨를 확인하고 만약을 위해 다 챙기죠. 문제 없는 순조로운 여행을 위해 떠나기 전에 일정을 재차 확인합니다." }, { type: "경험 (Experience)", text: "I once lost my wallet at an airport abroad. I was in **total panic** because all my cards and ID were in there. Luckily, a kind person found it and brought it to the information desk.", trans: "한번은 해외 공항에서 지갑을 잃어버린 적이 있어요. 모든 카드와 신분증이 거기 있어서 완전 멘붕이었죠. 다행히 친절한 분이 찾아서 안내 데스크에 갖다 주셨어요." }] },
  "집 휴가": { expected: ["Why do you prefer staying at home for vacation? What are the specific benefits?", "What do you usually do at home during your vacation? Describe a typical day.", "Tell me about a memorable staycation you had recently."], scripts: [{ type: "묘사 (Description)", text: "Actually, I'm a total **homebody**. I'm introverted, so being in crowded places drains my energy. Staying at home is the best way for me to **recharge** properly.", trans: "사실 전 완전 집순이예요. 내향적이라 사람 많은 곳에 있으면 기가 빨려요. 집에서 쉬는 게 저에겐 제대로 재충전하는 가장 좋은 방법이죠." }, { type: "루틴 (Routine)", text: "During my staycation, I usually **bingewatch** series on Netflix. I just love lying on my cozy couch with some snacks and doing nothing but watching movies all day.", trans: "집 휴가 기간에는 보통 넷플릭스를 정주행해요. 아늑한 소파에 누워서 간식과 함께 하루 종일 영화 보는 것 외에는 아무것도 안 하는 게 너무 좋아요." }, { type: "경험 (Experience)", text: "Last Christmas, I spent the whole day with my family at home. We cooked a huge steak dinner and played board games. It was much more **meaningful** than going out to a crowded restaurant.", trans: "지난 크리스마스에 가족들과 하루 종일 집에서 시간을 보냈어요. 스테이크 정찬을 요리하고 보드게임을 했죠. 북적이는 식당에 가는 것보다 훨씬 의미 있는 시간이었어요." }] }
};

const topics = Object.keys(masterData);

const socialIssueBank = [
  { q: "Nowadays, personal information security is becoming a serious concern globally. What do you think are the most effective ways to protect our online privacy?", topic: "개인정보 및 보안" },
  { q: "Fake news travels much faster than the truth on social media platforms today. How does misinformation affect our society and how can we distinguish it?", topic: "가짜뉴스 및 정보 신뢰도" },
  { q: "Many people find it difficult to stay away from their smartphones for even an hour. In your opinion, what are the long-term consequences of smartphone addiction on mental health?", topic: "스마트폰 및 미디어 중독" },
  { q: "The gap between digital experts and those who lack digital skills is widening among different generations. What kind of support should be provided to help elderly people adapt?", topic: "세대 간 디지털 격차" },
  { q: "Artificial intelligence is rapidly changing the way we work and live today. Do you believe that AI will create more opportunities for humans or eventually take away jobs?", topic: "AI와 자동화의 영향" }
];

const surveyToOpicTopic = {
  "영화보기": "movies",
  "공연보기": "concerts",
  "콘서트보기": "concerts",
  "공원가기": "parks",
  "캠핑하기": "camping",
  "음악 감상하기": "music",
  "조깅": "jogwalk",
  "걷기": "jogwalk",
  "집에서 보내는 휴가": "staycation",
  "국내 여행": "travel",
  "해외 여행": "travel",
};

const opicAiQuestionBank = [
  { id: 1, topic: "movies", en: "Describe your favorite movie theater. Where is it and what does it look like?", ko: "가장 좋아하는 영화관을 묘사하세요. 어디에 있고 어떤 모습인가요?" },
  { id: 2, topic: "movies", en: "What kind of movies do you usually watch and why?", ko: "주로 어떤 장르의 영화를 보며 그 이유는 무엇인가요?" },
  { id: 3, topic: "movies", en: "Tell me about the last movie you saw. Who did you go with?", ko: "최근에 본 영화에 대해 말해 주세요. 누구와 갔나요?" },
  { id: 4, topic: "movies", en: "How have movies changed from your childhood to now?", ko: "어린 시절과 비교해 영화가 어떻게 변했나요?" },
  { id: 5, topic: "movies", en: "Tell me about a memorable movie you saw recently. Why was it special?", ko: "최근 본 영화 중 기억에 남는 것은 무엇이며 왜 특별했나요?" },
  { id: 6, topic: "movies", en: "Describe the routine you follow when you go to the cinema.", ko: "영화관에 갈 때 하는 일련의 과정을 설명해 주세요." },
  { id: 7, topic: "movies", en: "Have you ever had an unexpected experience at a theater?", ko: "영화관에서 예상치 못한 경험을 한 적이 있나요?" },
  { id: 8, topic: "movies", en: "Compare your favorite movie genre with a genre you dislike.", ko: "좋아하는 장르와 싫어하는 장르를 비교해 보세요." },
  { id: 9, topic: "movies", en: "Who is your favorite actor or actress? Describe them.", ko: "가장 좋아하는 배우는 누구이며 그들을 묘사해 보세요." },
  { id: 10, topic: "movies", en: "Tell me about the issues or news related to the movie industry these days.", ko: "최근 영화 산업과 관련된 이슈나 뉴스에 대해 말해 보세요." },
  { id: 11, topic: "concerts", en: "What kind of concerts or performances do you like to attend?", ko: "어떤 종류의 콘서트나 공연에 가는 것을 좋아하나요?" },
  { id: 12, topic: "concerts", en: "Describe a concert venue you frequently visit.", ko: "자주 가는 콘서트홀이나 공연장 장소를 묘사해 주세요." },
  { id: 13, topic: "concerts", en: "Tell me about the first concert you ever went to.", ko: "처음으로 갔던 콘서트에 대해 이야기해 주세요." },
  { id: 14, topic: "concerts", en: "How do you usually get tickets for a performance?", ko: "공연 티켓을 보통 어떻게 예매하나요?" },
  { id: 15, topic: "concerts", en: "Tell me about a time you went to a performance that was better than expected.", ko: "기대보다 좋았던 공연에 대해 말해 주세요." },
  { id: 16, topic: "concerts", en: "Describe the atmosphere of a concert you recently attended.", ko: "최근 참석한 콘서트의 분위기를 묘사해 주세요." },
  { id: 17, topic: "concerts", en: "What are some things you do before and after a concert?", ko: "콘서트 전후로 무엇을 하나요?" },
  { id: 18, topic: "concerts", en: "Have you ever had a problem when attending a performance?", ko: "공연 관람 중 문제를 겪은 적이 있나요?" },
  { id: 19, topic: "concerts", en: "Compare attending a live performance versus watching a recording.", ko: "라이브 공연 관람과 영상 시청을 비교해 보세요." },
  { id: 20, topic: "concerts", en: "Talk about a famous artist or performer you are interested in.", ko: "관심 있는 유명 아티스트나 공연자에 대해 말해 주세요." },
  { id: 21, topic: "parks", en: "Describe a park you often visit. What do people do there?", ko: "자주 가는 공원을 묘사하세요. 사람들은 거기서 무엇을 하나요?" },
  { id: 22, topic: "parks", en: "When and with whom do you usually go to the park?", ko: "보통 언제, 누구와 공원에 가나요?" },
  { id: 23, topic: "parks", en: "Tell me about a memorable event that happened at a park.", ko: "공원에서 있었던 기억에 남는 사건을 말해 주세요." },
  { id: 24, topic: "parks", en: "How have parks changed over the last 10 years in your country?", ko: "지난 10년간 한국의 공원들이 어떻게 변했나요?" },
  { id: 25, topic: "parks", en: "Describe the activities you do for exercise when you are at a park.", ko: "공원에서 운동으로 하는 활동들을 설명해 주세요." },
  { id: 26, topic: "parks", en: "Talk about the weather when you last visited a park.", ko: "마지막으로 공원에 갔을 때 날씨가 어땠는지 말해 보세요." },
  { id: 27, topic: "parks", en: "Why do you think people go to parks these days?", ko: "요즘 사람들이 왜 공원에 간다고 생각하나요?" },
  { id: 28, topic: "parks", en: "Describe a park you visited during your childhood.", ko: "어린 시절에 방문했던 공원을 묘사해 보세요." },
  { id: 29, topic: "parks", en: "Compare two different parks you have visited.", ko: "방문해 본 두 공원의 차이점을 비교해 보세요." },
  { id: 30, topic: "parks", en: "What are some problems or issues people face in public parks?", ko: "공공 공원에서 사람들이 겪는 문제나 이슈는 무엇인가요?" },
  { id: 31, topic: "camping", en: "Where do you usually go camping and why do you like that place?", ko: "주로 어디로 캠핑을 가며 왜 그곳을 좋아하나요?" },
  { id: 32, topic: "camping", en: "What items do you usually pack for a camping trip?", ko: "캠핑을 갈 때 보통 어떤 물건들을 챙기나요?" },
  { id: 33, topic: "camping", en: "Describe your most recent camping trip in detail.", ko: "가장 최근에 다녀온 캠핑에 대해 자세히 묘사해 주세요." },
  { id: 34, topic: "camping", en: "Tell me about the first time you ever went camping.", ko: "처음 캠핑을 갔던 경험에 대해 말해 주세요." },
  { id: 35, topic: "camping", en: "What is the most important rule to follow when camping?", ko: "캠핑할 때 지켜야 할 가장 중요한 규칙은 무엇인가요?" },
  { id: 36, topic: "camping", en: "Tell me about a time you had a problem with the weather while camping.", ko: "캠핑 중 날씨 때문에 겪었던 문제에 대해 말해 보세요." },
  { id: 37, topic: "camping", en: "How has camping changed compared to the past?", ko: "과거와 비교해 캠핑 문화가 어떻게 변했나요?" },
  { id: 38, topic: "camping", en: "Describe the food you usually cook while camping.", ko: "캠핑 중 주로 요리해 먹는 음식에 대해 설명해 주세요." },
  { id: 39, topic: "camping", en: "What are the pros and cons of camping versus staying in a hotel?", ko: "호텔 숙박과 비교한 캠핑의 장단점은 무엇인가요?" },
  { id: 40, topic: "camping", en: "Tell me about an unforgettable experience you had while camping.", ko: "캠핑 중 잊지 못할 경험에 대해 말해 주세요." },
  { id: 41, topic: "music", en: "What genres of music do you like and who is your favorite singer?", ko: "어떤 장르를 좋아하며 좋아하는 가수는 누구인가요?" },
  { id: 42, topic: "music", en: "When and where do you usually listen to music?", ko: "보통 언제 어디서 음악을 듣나요?" },
  { id: 43, topic: "music", en: "Describe how your musical taste has changed over time.", ko: "음악적 취향이 시간에 따라 어떻게 변했는지 묘사해 주세요." },
  { id: 44, topic: "music", en: "How did you first become interested in music?", ko: "처음 음악에 관심을 갖게 된 계기는 무엇인가요?" },
  { id: 45, topic: "music", en: "Talk about the devices you use to listen to music.", ko: "음악을 들을 때 사용하는 기기들에 대해 말해 주세요." },
  { id: 46, topic: "music", en: "Have you ever been to a live music event? Describe it.", ko: "라이브 음악 행사에 간 적이 있나요? 묘사해 주세요." },
  { id: 47, topic: "music", en: "Compare the music you liked as a child to the music you like now.", ko: "어릴 때 좋아한 음악과 지금 좋아하는 음악을 비교하세요." },
  { id: 48, topic: "music", en: "Why do you think music is important to people?", ko: "왜 음악이 사람들에게 중요하다고 생각하나요?" },
  { id: 49, topic: "music", en: "Tell me about a time music helped you through a difficult situation.", ko: "음악이 힘든 상황을 극복하는 데 도움이 되었던 때를 말해 주세요." },
  { id: 50, topic: "music", en: "What is a popular music trend in your country right now?", ko: "현재 한국에서 유행하는 음악 트렌드는 무엇인가요?" },
  { id: 51, topic: "jogwalk", en: "Where do you usually go for a walk or jog?", ko: "주로 어디로 걷기나 조깅을 하러 가나요?" },
  { id: 52, topic: "jogwalk", en: "Describe your typical routine when you go out for a jog.", ko: "조깅하러 나갈 때의 전형적인 루틴을 묘사해 주세요." },
  { id: 53, topic: "jogwalk", en: "What do you wear or carry when you go jogging?", ko: "조깅할 때 무엇을 입거나 챙겨가나요?" },
  { id: 54, topic: "jogwalk", en: "Tell me about a time you went walking and saw something interesting.", ko: "걷다가 흥미로운 것을 본 경험을 말해 주세요." },
  { id: 55, topic: "jogwalk", en: "Why did you start jogging or walking for exercise?", ko: "운동으로 조깅이나 걷기를 시작한 이유는 무엇인가요?" },
  { id: 56, topic: "jogwalk", en: "How do you feel after a long walk or a jog?", ko: "오래 걷거나 조깅한 후 기분이 어떤가요?" },
  { id: 57, topic: "jogwalk", en: "Talk about a special place you discovered while walking.", ko: "걷다가 발견한 특별한 장소에 대해 말해 주세요." },
  { id: 58, topic: "jogwalk", en: "Compare jogging in the morning versus jogging in the evening.", ko: "아침 조깅과 저녁 조깅을 비교해 보세요." },
  { id: 59, topic: "jogwalk", en: "Have you ever had an injury or physical problem while jogging?", ko: "조깅 중 부상을 당하거나 신체적 문제를 겪은 적이 있나요?" },
  { id: 60, topic: "jogwalk", en: "How has your fitness improved since you started walking regularly?", ko: "규칙적으로 걷기 시작한 후 체력이 어떻게 좋아졌나요?" },
  { id: 61, topic: "staycation", en: "Describe what you typically do when you spend a vacation at home.", ko: "집에서 휴가를 보낼 때 주로 무엇을 하는지 묘사해 주세요." },
  { id: 62, topic: "staycation", en: "Why do you sometimes choose to stay home instead of traveling?", ko: "여행 대신 집에서 쉬는 것을 선택하는 이유는 무엇인가요?" },
  { id: 63, topic: "staycation", en: "Tell me about the most recent vacation you spent at home.", ko: "가장 최근에 집에서 보낸 휴가에 대해 말해 주세요." },
  { id: 64, topic: "staycation", en: "What are the advantages of spending your vacation at home?", ko: "집에서 휴가를 보내는 것의 장점은 무엇인가요?" },
  { id: 65, topic: "staycation", en: "Describe a specific hobby you enjoy during your home vacation.", ko: "집에서 휴가 동안 즐기는 특정 취미를 묘사해 주세요." },
  { id: 66, topic: "staycation", en: "How do you prepare your home to make it feel like a vacation spot?", ko: "집을 휴가지처럼 느끼게 하기 위해 어떻게 준비하나요?" },
  { id: 67, topic: "staycation", en: "Tell me about a memorable meal you had during a stay-at-home vacation.", ko: "집 휴가 중 먹었던 기억에 남는 식사에 대해 말해 주세요." },
  { id: 68, topic: "staycation", en: "Compare a vacation at home with a vacation abroad.", ko: "집 휴가와 해외 여행 휴가를 비교해 보세요." },
  { id: 69, topic: "staycation", en: "What do you usually watch or read when relaxing at home?", ko: "집에서 쉴 때 주로 무엇을 보거나 읽나요?" },
  { id: 70, topic: "staycation", en: "Have you ever had a plan for a home vacation that didn't go as expected?", ko: "집 휴가 계획이 예상대로 되지 않았던 적이 있나요?" },
  { id: 71, topic: "travel", en: "Describe a domestic travel destination you recently visited.", ko: "최근 방문한 국내 여행지를 묘사해 주세요." },
  { id: 72, topic: "travel", en: "What are the main differences between domestic and overseas travel?", ko: "국내 여행과 해외 여행의 주요 차이점은 무엇인가요?" },
  { id: 73, topic: "travel", en: "Tell me about your first trip abroad. Where did you go?", ko: "처음으로 간 해외 여행에 대해 말해 주세요. 어디였나요?" },
  { id: 74, topic: "travel", en: "What is the most important thing you consider when planning a trip?", ko: "여행 계획을 세울 때 가장 중요하게 생각하는 것은 무엇인가요?" },
  { id: 75, topic: "travel", en: "Describe a memorable person you met while traveling.", ko: "여행 중 만난 기억에 남는 사람을 묘사해 주세요." },
  { id: 76, topic: "travel", en: "Tell me about a time you experienced a flight delay or cancellation.", ko: "비행기 지연이나 취소 경험에 대해 말해 주세요." },
  { id: 77, topic: "travel", en: "What kind of food do you like to try when you travel?", ko: "여행할 때 어떤 음식을 먹어보는 것을 좋아하나요?" },
  { id: 78, topic: "travel", en: "Talk about a travel destination you want to visit in the future.", ko: "미래에 방문하고 싶은 여행지에 대해 말해 보세요." },
  { id: 79, topic: "travel", en: "How do you usually pack for a long-distance trip?", ko: "장거리 여행을 위해 보통 짐을 어떻게 싸나요?" },
  { id: 80, topic: "travel", en: "Tell me about a problem you faced with your accommodation.", ko: "숙소에서 겪었던 문제에 대해 말해 주세요." },
  { id: 81, topic: "travel", en: "Describe the most beautiful scenery you have ever seen while traveling.", ko: "여행 중 본 가장 아름다운 풍경을 묘사해 보세요." },
  { id: 82, topic: "travel", en: "Why do you think people travel to other countries?", ko: "사람들이 왜 다른 나라로 여행을 간다고 생각하나요?" },
  { id: 83, topic: "travel", en: "Compare the transportation you use during domestic versus overseas trips.", ko: "국내 여행과 해외 여행 시 사용하는 교통수단을 비교하세요." },
  { id: 84, topic: "travel", en: "Tell me about a time you got lost while traveling.", ko: "여행 중 길을 잃었던 경험을 말해 주세요." },
  { id: 85, topic: "travel", en: "What are some essential travel apps you use?", ko: "사용하는 필수 여행 앱들은 무엇인가요?" },
  { id: 86, topic: "roleplay", en: "[Roleplay] Call a movie theater to ask about ticket prices and showtimes.", ko: "[롤플레이] 영화관에 전화해 티켓 가격과 상영 시간을 문의하세요." },
  { id: 87, topic: "roleplay", en: "[Roleplay] You bought a concert ticket but cannot go. Call a friend to offer it.", ko: "[롤플레이] 콘서트 티켓을 샀는데 못 가게 되었습니다. 친구에게 전화해 제안하세요." },
  { id: 88, topic: "roleplay", en: "[Roleplay] You are at a park and want to rent a bike. Ask the staff 3-4 questions.", ko: "[롤플레이] 공원에서 자전거를 빌리고 싶습니다. 직원에게 3~4가지 질문을 하세요." },
  { id: 89, topic: "roleplay", en: "[Roleplay] Call a camping site to make a reservation and ask about facilities.", ko: "[롤플레이] 캠핑장에 전화해 예약하고 시설에 대해 문의하세요." },
  { id: 90, topic: "roleplay", en: "[Roleplay] You are at a music store. Ask the clerk about the latest albums.", ko: "[롤플레이] 음악 매장에서 점원에게 최신 앨범에 대해 물어보세요." },
  { id: 91, topic: "roleplay", en: "[Roleplay] A friend invited you to go jogging, but you are sick. Explain and suggest an alternative.", ko: "[롤플레이] 친구가 조깅 가자는데 아픕니다. 설명하고 대안을 제시하세요." },
  { id: 92, topic: "roleplay", en: "[Roleplay] You planned a trip abroad but your passport expired. Call the travel agency.", ko: "[롤플레이] 해외 여행을 계획했는데 여권이 만료되었습니다. 여행사에 전화하세요." },
  { id: 93, topic: "roleplay", en: "[Roleplay] Ask a friend who loves traveling for recommendations for your next trip.", ko: "[롤플레이] 여행을 좋아하는 친구에게 다음 여행지 추천을 부탁하며 질문하세요." },
  { id: 94, topic: "roleplay", en: "[Roleplay] You are at a hotel check-in desk, but they can't find your reservation. Solve the issue.", ko: "[롤플레이] 호텔 체크인 중 예약이 확인되지 않습니다. 문제를 해결하세요." },
  { id: 95, topic: "roleplay", en: "[Roleplay] Call a friend and suggest staying at home together for the upcoming holiday.", ko: "[롤플레이] 친구에게 전화해 이번 연휴에 같이 집에서 쉬자고 제안하세요." },
  { id: 96, topic: "advanced", en: "Compare the way people traveled in the past to how they travel today.", ko: "과거와 현재의 여행 방식을 비교하세요." },
  { id: 97, topic: "advanced", en: "What are some environmental issues caused by camping or tourism?", ko: "캠핑이나 관광으로 인한 환경 문제는 무엇인가요?" },
  { id: 98, topic: "advanced", en: "Talk about a time when technology changed your experience of a hobby.", ko: "기술이 당신의 취미 생활을 바꾼 경험에 대해 말해 보세요." },
  { id: 99, topic: "advanced", en: "Describe a time you had an unexpected conversation with a stranger during a trip.", ko: "여행 중 낯선 사람과 예상치 못한 대화를 나눈 경험을 말해 주세요." },
  { id: 100, topic: "advanced", en: "How has the way we consume music and movies changed due to the internet?", ko: "인터넷으로 인해 음악과 영화를 소비하는 방식이 어떻게 변했나요?" },
];

const fillers = [
  { phrase: "That's an interesting question.", meaning: "흥미로운 질문이네요.", cat: "질문 리액션", isBest: true, ex: "That's an interesting question. I've never really thought about it that way before." },
  { phrase: "You know, I was actually expecting this question.", meaning: "있잖아요, 사실 이 질문 나올 줄 알았어요.", cat: "질문 리액션", isBest: true, ex: "You know, I was actually expecting this question because it's a popular topic." },
  { phrase: "Oh, where should I even start?", meaning: "오, 어디서부터 시작해야 할까요?", cat: "질문 리액션", isBest: true, ex: "Oh, where should I even start? There are so many things I could say." },
  { phrase: "To be honest, I'm a bit put on the spot here.", meaning: "솔직히 말씀드리면, 지금 좀 당황스럽네요.", cat: "질문 리액션", isBest: true, ex: "To be honest, I'm a bit put on the spot here. Let me think for a second." },
  { phrase: "I'm glad you asked that.", meaning: "그 질문을 해주셔서 기쁘네요.", cat: "질문 리액션", isBest: true, ex: "I'm glad you asked that. I actually have a clear answer." },
  { phrase: "That's a great question.", meaning: "좋은 질문이네요.", cat: "질문 리액션", isBest: false, ex: "That's a great question. I think it depends on the situation." },
  { phrase: "Honestly, I've thought about this before.", meaning: "솔직히 이거 예전부터 생각해본 적 있어요.", cat: "질문 리액션", isBest: false, ex: "Honestly, I've thought about this before, especially when I was in college." },
  { phrase: "I have a pretty clear opinion on that.", meaning: "그건 제 생각이 꽤 명확해요.", cat: "질문 리액션", isBest: false, ex: "I have a pretty clear opinion on that. I prefer staying in." },
  { phrase: "That brings back memories.", meaning: "그 얘기 들으니까 추억이 떠오르네요.", cat: "질문 리액션", isBest: false, ex: "That brings back memories. I used to do that all the time as a kid." },
  { phrase: "I've got a quick story about that.", meaning: "그거 관련해서 짧은 일화가 있어요.", cat: "질문 리액션", isBest: false, ex: "I've got a quick story about that. It happened last weekend." },
  { phrase: "I can totally relate to that question.", meaning: "그 질문에 완전 공감돼요.", cat: "질문 리액션", isBest: false, ex: "I can totally relate to that question because I deal with it often." },
  { phrase: "That’s something I’ve noticed too.", meaning: "저도 그거 느꼈던 부분이에요.", cat: "질문 리액션", isBest: false, ex: "That’s something I’ve noticed too, especially these days." },
  { phrase: "Let me start with the big picture.", meaning: "큰 틀부터 말씀드릴게요.", cat: "질문 리액션", isBest: false, ex: "Let me start with the big picture. Then I'll give you details." },
  { phrase: "I’ve never been asked that before.", meaning: "그런 질문은 처음 받아보네요.", cat: "질문 리액션", isBest: false, ex: "I’ve never been asked that before, but it’s a fun one." },
  { phrase: "That’s a tricky one.", meaning: "그건 좀 어려운 질문이네요.", cat: "질문 리액션", isBest: false, ex: "That’s a tricky one. I need a moment to think." },
  { phrase: "How can I put this?", meaning: "이걸 어떻게 말해야 할까요?", cat: "시간 벌기", isBest: true, ex: "It was... how can I put this? Kind of overwhelming." },
  { phrase: "What's the word I'm looking for?", meaning: "제가 찾으려는 단어가 뭐였죠?", cat: "시간 벌기", isBest: true, ex: "The view was... what's the word I'm looking for? Breathtaking." },
  { phrase: "Let me gather my thoughts for a moment.", meaning: "잠시 생각 좀 정리할게요.", cat: "시간 벌기", isBest: true, ex: "Let me gather my thoughts for a moment. Okay, here’s what happened." },
  { phrase: "Give me a second.", meaning: "잠시만요.", cat: "시간 벌기", isBest: true, ex: "Give me a second. I'm trying to remember the exact time." },
  { phrase: "Let me think for a second.", meaning: "잠깐만 생각해볼게요.", cat: "시간 벌기", isBest: true, ex: "Let me think for a second. Yeah, I usually go there on weekends." },
  { phrase: "Hmm...", meaning: "흠...", cat: "시간 벌기", isBest: false, ex: "Hmm... I guess I'd say it's pretty convenient." },
  { phrase: "Let's see...", meaning: "어디 보자...", cat: "시간 벌기", isBest: false, ex: "Let's see... the last time was about two months ago." },
  { phrase: "If I recall correctly,", meaning: "제가 기억하기로는,", cat: "시간 벌기", isBest: false, ex: "If I recall correctly, it was raining that day." },
  { phrase: "If my memory serves me right,", meaning: "제 기억이 맞다면,", cat: "시간 벌기", isBest: false, ex: "If my memory serves me right, it started around 6 p.m." },
  { phrase: "I’m trying to remember the details.", meaning: "디테일이 기억나려고 하는 중이에요.", cat: "시간 벌기", isBest: false, ex: "I’m trying to remember the details. It was either Tuesday or Wednesday." },
  { phrase: "I'm drawing a blank.", meaning: "머릿속이 하얘졌어요.", cat: "시간 벌기", isBest: false, ex: "I'm drawing a blank on the name, but I remember the place clearly." },
  { phrase: "It's on the tip of my tongue.", meaning: "입가에서 맴도는데요.", cat: "시간 벌기", isBest: false, ex: "It's on the tip of my tongue. The café was called... 'Green Bean'!" },
  { phrase: "It slipped my mind for a second.", meaning: "잠깐 깜빡했네요.", cat: "시간 벌기", isBest: false, ex: "It slipped my mind for a second, but now I remember." },
  { phrase: "I might be mixing things up.", meaning: "제가 헷갈리고 있는 걸 수도 있어요.", cat: "시간 벌기", isBest: false, ex: "I might be mixing things up, but I think it happened last year." },
  { phrase: "Bear with me for a moment.", meaning: "잠깐만 기다려주세요.", cat: "시간 벌기", isBest: false, ex: "Bear with me for a moment. I'm organizing it in my head." },
  { phrase: "Let me walk you through it.", meaning: "차근차근 설명해볼게요.", cat: "시간 벌기", isBest: false, ex: "Let me walk you through it. First, I got ready, then I left." },
  { phrase: "Where was I?", meaning: "제가 어디까지 말했죠?", cat: "시간 벌기", isBest: false, ex: "Where was I? Oh right—then we took a taxi." },
  { phrase: "What else was there?", meaning: "또 뭐가 있었더라?", cat: "시간 벌기", isBest: false, ex: "We had snacks and drinks... what else was there?" },
  { phrase: "Let me put it this way.", meaning: "이렇게 말해볼게요.", cat: "시간 벌기", isBest: false, ex: "Let me put it this way. It was good, but not amazing." },
  { phrase: "I need a second to piece it together.", meaning: "정리할 시간이 조금 필요해요.", cat: "시간 벌기", isBest: false, ex: "I need a second to piece it together. Okay, now I’ve got it." },
  { phrase: "To be completely honest with you,", meaning: "정말 솔직히 말씀드리면,", cat: "감정 및 강조", isBest: true, ex: "To be completely honest with you, I wasn't a big fan of it." },
  { phrase: "Believe it or not,", meaning: "믿으실지 모르겠지만,", cat: "감정 및 강조", isBest: true, ex: "Believe it or not, I've never tried it before." },
  { phrase: "Frankly speaking,", meaning: "솔직히 말해서,", cat: "감정 및 강조", isBest: true, ex: "Frankly speaking, I think it's a waste of time." },
  { phrase: "Without a doubt,", meaning: "의심의 여지 없이,", cat: "감정 및 강조", isBest: true, ex: "Without a doubt, it was the best decision I've made." },
  { phrase: "I was honestly blown away.", meaning: "진짜로 충격/감동받았어요.", cat: "감정 및 강조", isBest: true, ex: "I was honestly blown away by how good the food was." },
  { phrase: "It really made my day.", meaning: "진짜 기분 좋아졌어요.", cat: "감정 및 강조", isBest: false, ex: "The compliment really made my day." },
  { phrase: "I was over the moon.", meaning: "너무 행복했어요.", cat: "감정 및 강조", isBest: false, ex: "I was over the moon when I passed the test." },
  { phrase: "I was so frustrated.", meaning: "너무 답답했어요.", cat: "감정 및 강조", isBest: false, ex: "I was so frustrated because nothing was working." },
  { phrase: "I couldn't believe my eyes.", meaning: "제 눈을 의심했어요.", cat: "감정 및 강조", isBest: false, ex: "I couldn't believe my eyes when I saw the view." },
  { phrase: "It was such a relief.", meaning: "정말 안도했어요.", cat: "감정 및 강조", isBest: false, ex: "It was such a relief to finally get home." },
  { phrase: "I was genuinely impressed.", meaning: "진심으로 인상 깊었어요.", cat: "감정 및 강조", isBest: false, ex: "I was genuinely impressed by their kindness." },
  { phrase: "It was a bit overwhelming.", meaning: "조금 버거웠어요.", cat: "감정 및 강조", isBest: false, ex: "It was a bit overwhelming at first, but I got used to it." },
  { phrase: "I was totally in shock.", meaning: "완전 충격이었어요.", cat: "감정 및 강조", isBest: false, ex: "I was totally in shock when I heard the news." },
  { phrase: "It felt like a dream.", meaning: "꿈같았어요.", cat: "감정 및 강조", isBest: false, ex: "It felt like a dream to be there in person." },
  { phrase: "I was a nervous wreck.", meaning: "너무 긴장했어요.", cat: "감정 및 강조", isBest: false, ex: "I was a nervous wreck before the presentation." },
  { phrase: "It was so satisfying.", meaning: "너무 속 시원했어요.", cat: "감정 및 강조", isBest: false, ex: "It was so satisfying to finish everything on time." },
  { phrase: "I was really touched.", meaning: "감동받았어요.", cat: "감정 및 강조", isBest: false, ex: "I was really touched by what she said." },
  { phrase: "Honestly, it was a lifesaver.", meaning: "솔직히 그거 덕분에 살았어요.", cat: "감정 및 강조", isBest: false, ex: "Honestly, the GPS was a lifesaver." },
  { phrase: "It was totally worth it.", meaning: "완전 그럴 가치가 있었어요.", cat: "감정 및 강조", isBest: false, ex: "It was totally worth it, even though it was expensive." },
  { phrase: "At the end of the day,", meaning: "결국에는,", cat: "감정 및 강조", isBest: false, ex: "At the end of the day, health matters the most." },
  { phrase: "What I'm trying to say is,", meaning: "제가 말하고자 하는 핵심은,", cat: "설명 및 정정", isBest: true, ex: "What I'm trying to say is, it's more about balance than speed." },
  { phrase: "Let me rephrase that.", meaning: "표현을 좀 바꿔볼게요.", cat: "설명 및 정정", isBest: true, ex: "Let me rephrase that. It wasn't difficult—just time-consuming." },
  { phrase: "To put it simply,", meaning: "단순히 말하자면,", cat: "설명 및 정정", isBest: true, ex: "To put it simply, I prefer quiet places." },
  { phrase: "In other words,", meaning: "다시 말해서,", cat: "설명 및 정정", isBest: true, ex: "In other words, it was a complete mess." },
  { phrase: "Let me clarify that.", meaning: "그 부분을 명확히 할게요.", cat: "설명 및 정정", isBest: true, ex: "Let me clarify that. I wasn't angry—I was just tired." },
  { phrase: "To be more specific,", meaning: "더 구체적으로는,", cat: "설명 및 정정", isBest: false, ex: "To be more specific, I go there every Friday night." },
  { phrase: "To be more precise,", meaning: "더 정확히는,", cat: "설명 및 정정", isBest: false, ex: "To be more precise, it happened around 7:30." },
  { phrase: "What I mean is,", meaning: "제 말은 그러니까,", cat: "설명 및 정정", isBest: false, ex: "What I mean is, I like it, but I don't love it." },
  { phrase: "That’s not exactly what I meant.", meaning: "제 의도는 그게 정확히 아니었어요.", cat: "설명 및 정정", isBest: false, ex: "That’s not exactly what I meant. I meant we should go tomorrow." },
  { phrase: "Let me back up a bit.", meaning: "잠깐만 앞부분부터 다시 말할게요.", cat: "설명 및 정정", isBest: false, ex: "Let me back up a bit. The real issue started earlier." },
  { phrase: "I’m getting ahead of myself.", meaning: "제가 너무 앞서갔네요.", cat: "설명 및 정정", isBest: false, ex: "I’m getting ahead of myself. Let me explain the background first." },
  { phrase: "I think I’m getting sidetracked.", meaning: "이야기가 좀 샜네요.", cat: "설명 및 정정", isBest: false, ex: "I think I’m getting sidetracked. Anyway, back to the question." },
  { phrase: "Does that make sense?", meaning: "이해가 되시나요?", cat: "설명 및 정정", isBest: false, ex: "It's kind of a tradition in my family. Does that make sense?" },
  { phrase: "Let me give you an example.", meaning: "예시를 들어볼게요.", cat: "설명 및 정정", isBest: false, ex: "Let me give you an example. Like, I always prep my clothes the night before." },
  { phrase: "So, to be clear,", meaning: "정리하자면,", cat: "설명 및 정정", isBest: false, ex: "So, to be clear, I don't go there often—maybe once a month." },
  { phrase: "Speaking of which,", meaning: "말이 나온 김에,", cat: "전환 및 추가", isBest: true, ex: "Speaking of which, I actually went there last weekend." },
  { phrase: "By the way,", meaning: "그나저나,", cat: "전환 및 추가", isBest: true, ex: "By the way, there's a great café near my place." },
  { phrase: "Anyway, back to the point,", meaning: "어쨌든 다시 본론으로,", cat: "전환 및 추가", isBest: true, ex: "Anyway, back to the point, I usually go there after work." },
  { phrase: "On top of that,", meaning: "게다가,", cat: "전환 및 추가", isBest: true, ex: "On top of that, the weather was perfect." },
  { phrase: "Another thing is,", meaning: "또 다른 점은,", cat: "전환 및 추가", isBest: true, ex: "Another thing is, the place is super easy to get to." },
  { phrase: "Also,", meaning: "그리고 또,", cat: "전환 및 추가", isBest: false, ex: "Also, I like the fact that it's quiet." },
  { phrase: "Plus,", meaning: "게다가,", cat: "전환 및 추가", isBest: false, ex: "Plus, it doesn't cost much." },
  { phrase: "Besides that,", meaning: "그거 말고도,", cat: "전환 및 추가", isBest: false, ex: "Besides that, I meet my friends there sometimes." },
  { phrase: "Not to mention,", meaning: "~은 말할 것도 없고,", cat: "전환 및 추가", isBest: false, ex: "Not to mention, the staff are really friendly." },
  { phrase: "That reminds me,", meaning: "그거 보니까 생각났는데,", cat: "전환 및 추가", isBest: false, ex: "That reminds me, I should go there again soon." },
  { phrase: "As for me,", meaning: "저 같은 경우는,", cat: "전환 및 추가", isBest: false, ex: "As for me, I prefer staying at home." },
  { phrase: "In my case,", meaning: "제 경우에는,", cat: "전환 및 추가", isBest: false, ex: "In my case, mornings are the best time to study." },
  { phrase: "For example,", meaning: "예를 들어,", cat: "전환 및 추가", isBest: false, ex: "For example, I always listen to music while commuting." },
  { phrase: "In addition,", meaning: "추가로,", cat: "전환 및 추가", isBest: false, ex: "In addition, I try to keep a simple routine." },
  { phrase: "Last but not least,", meaning: "마지막으로 중요한 건,", cat: "전환 및 추가", isBest: false, ex: "Last but not least, it helps me reduce stress." },
  { phrase: "So yeah, that's pretty much it.", meaning: "그래서 뭐, 대충 그게 다예요.", cat: "깔끔한 마무리", isBest: true, ex: "So yeah, that's pretty much it. That's why I like it so much." },
  { phrase: "Anyway, that wraps it up.", meaning: "어쨌든, 이걸로 마무리할게요.", cat: "깔끔한 마무리", isBest: true, ex: "Anyway, that wraps it up. Thanks for asking." },
  { phrase: "Long story short,", meaning: "긴 이야기를 짧게 하자면,", cat: "깔끔한 마무리", isBest: true, ex: "Long story short, everything worked out in the end." },
  { phrase: "All in all,", meaning: "전반적으로,", cat: "깔끔한 마무리", isBest: true, ex: "All in all, it was a great experience." },
  { phrase: "That’s about it.", meaning: "그게 다예요.", cat: "깔끔한 마무리", isBest: true, ex: "That’s about it. I think I covered everything." },
  { phrase: "I think that covers everything.", meaning: "이걸로 다 말씀드린 것 같아요.", cat: "깔끔한 마무리", isBest: false, ex: "I think that covers everything about my routine." },
  { phrase: "I guess that's everything.", meaning: "이게 전부인 것 같네요.", cat: "깔끔한 마무리", isBest: false, ex: "I guess that's everything. That's my honest answer." },
  { phrase: "To sum up,", meaning: "요약하자면,", cat: "깔끔한 마무리", isBest: false, ex: "To sum up, it was simple, relaxing, and fun." },
  { phrase: "In a nutshell,", meaning: "한마디로,", cat: "깔끔한 마무리", isBest: false, ex: "In a nutshell, I love it because it's convenient." },
  { phrase: "That’s the whole story.", meaning: "그게 이야기의 전부예요.", cat: "깔끔한 마무리", isBest: false, ex: "That’s the whole story. It was pretty unforgettable." },
  { phrase: "Hopefully that answers your question.", meaning: "제 답변이 도움이 되었길 바라요.", cat: "깔끔한 마무리", isBest: false, ex: "Hopefully that answers your question. That's my perspective." },
  { phrase: "I’ll leave it at that.", meaning: "여기까지 말씀드릴게요.", cat: "깔끔한 마무리", isBest: false, ex: "I’ll leave it at that. I don't want to overcomplicate it." },
  { phrase: "And that's why I feel that way.", meaning: "그래서 제가 그렇게 생각해요.", cat: "깔끔한 마무리", isBest: false, ex: "And that's why I feel that way about staying at home." },
  { phrase: "That’s all I can think of right now.", meaning: "지금 떠오르는 건 이게 전부예요.", cat: "깔끔한 마무리", isBest: false, ex: "That’s all I can think of right now. Maybe I’ll remember more later." },
  { phrase: "So, yeah—end of story.", meaning: "그래요—이게 끝이에요.", cat: "깔끔한 마무리", isBest: false, ex: "So, yeah—end of story. It was a funny moment." },
];

const vocabMap = {
  "집 휴가": {
    묘사: [{ w: "homebody", t: "집순이/집돌이" }, { w: "introverted", t: "내향적인" }, { w: "draining", t: "기 빨리는", emotion: true }, { w: "overstimulating", t: "자극 과다인" , emotion: true }, { w: "recharge", t: "재충전하다" }, { w: "unwind", t: "긴장 풀다" }, { w: "decompress", t: "머리 식히다" }, { w: "peace of mind", t: "마음의 평화", emotion: true }, { w: "comfort zone", t: "편한 영역" }, { w: "low-key", t: "조용하게" }, { w: "staycation", t: "스테이케이션" }, { w: "downtime", t: "휴식 시간" }],
    루틴: [{ w: "binge-watch", t: "정주행하다" }, { w: "grab snacks", t: "간식 챙기다" }, { w: "cozy", t: "아늑한" }, { w: "curl up", t: "웅크리고 눕다" }, { w: "take a nap", t: "낮잠 자다" }, { w: "stay in", t: "집에 있다" }, { w: "recharge properly", t: "제대로 재충전" }, { w: "guilt-free", t: "죄책감 없는", emotion: true }, { w: "fully relaxed", t: "완전 편안한", emotion: true }, { w: "routine", t: "루틴" }, { w: "order-in", t: "배달 시키다" }, { w: "catch up on sleep", t: "밀린 잠 자다" }],
    경험: [{ w: "meaningful", t: "의미 있는", emotion: true }, { w: "quality time", t: "소중한 시간" }, { w: "cook a big meal", t: "큰 요리하다" }, { w: "board games", t: "보드게임" }, { w: "bond with", t: "~와 유대" }, { w: "memorable", t: "기억에 남는", emotion: true }, { w: "unforgettable", t: "잊을 수 없는", emotion: true }, { w: "warm atmosphere", t: "따뜻한 분위기" }, { w: "crowded", t: "붐비는" }, { w: "prefer A to B", t: "A를 B보다 선호" }, { w: "celebration", t: "축하" }, { w: "special occasion", t: "특별한 경우" }],
  },
  "공원 가기": {
    묘사: [{ w: "picturesque", t: "그림 같은" }, { w: "peaceful", t: "평화로운" }, { w: "lively", t: "활기찬" }, { w: "scenery", t: "풍경" }, { w: "greenery", t: "녹지" }, { w: "fresh air", t: "신선한 공기" }, { w: "tranquil", t: "고요한", emotion: true }, { w: "serene", t: "잔잔한", emotion: true }, { w: "refreshing", t: "상쾌한", emotion: true }, { w: "vibe", t: "분위기" }, { w: "urban oasis", t: "도심 속 오아시스" }, { w: "scenic beauty", t: "자연의 아름다움" }],
    루틴: [{ w: "stretching", t: "스트레칭" }, { w: "take a walk", t: "산책하다" }, { w: "do laps", t: "몇 바퀴 돌다" }, { w: "listen to music", t: "음악 듣다" }, { w: "unwind", t: "긴장 풀다" }, { w: "clear my mind", t: "머리 비우다" }, { w: "on weekends", t: "주말에" }, { w: "habit", t: "습관" }, { w: "energized", t: "활력 있는", emotion: true }, { w: "relaxed", t: "편안한", emotion: true }, { w: "stroll around", t: "거닐다" }, { w: "outdoor activity", t: "야외 활동" }],
    경험: [{ w: "memorable", t: "기억에 남는", emotion: true }, { w: "unexpected", t: "예상치 못한" }, { w: "ran into", t: "우연히 마주치다" }, { w: "embarrassing", t: "창피한", emotion: true }, { w: "awkward", t: "어색한" , emotion: true }, { w: "pretend not to", t: "~인 척하다" }, { w: "walk away", t: "자리를 뜨다" }, { w: "couldn't believe", t: "믿기 힘들었다", emotion: true }, { w: "all of a sudden", t: "갑자기" }, { w: "detail", t: "디테일" }, { w: "out of the blue", t: "갑작스럽게" }, { w: "incident", t: "사건" }],
  },
};

// --- 2. Shared Global Helpers ---

const ttsCache = new Map();
const globalAudio = new Audio();

const fetchAndCacheTTS = async (textToCache) => {
  if (!textToCache || ttsCache.has(textToCache)) return;
  try {
    const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=${apiKey}`, {
      method: "POST", headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ contents: [{ parts: [{ text: "Read this naturally: " + textToCache }] }], generationConfig: { responseModalities: ["AUDIO"], speechConfig: { voiceConfig: { prebuiltVoiceConfig: { voiceName: "Kore" } } } }, model: "gemini-2.5-flash-preview-tts" })
    });
    const result = await response.json();
    const audioData = result.candidates?.[0]?.content?.parts?.[0]?.inlineData?.data;
    if (audioData) {
      const binaryString = window.atob(audioData);
      const bytes = new Uint8Array(binaryString.length);
      for (let i = 0; i < binaryString.length; i++) bytes[i] = binaryString.charCodeAt(i);
      const wavHeader = new ArrayBuffer(44);
      const view = new DataView(wavHeader);
      const sampleRate = 24000;
      view.setUint32(0, 0x52494646, false); view.setUint32(4, 36 + bytes.length, true);
      view.setUint32(8, 0x57415645, false); view.setUint32(12, 0x666d7420, false);
      view.setUint16(16, 16, true); view.setUint16(20, 1, true);
      view.setUint16(22, 1, true); view.setUint32(24, sampleRate, true);
      view.setUint32(28, sampleRate * 2, true); view.setUint16(32, 2, true);
      view.setUint16(34, 16, true); view.setUint32(36, 0x64617461, false);
      view.setUint32(40, bytes.length, true);
      const blob = new Blob([wavHeader, bytes], { type: 'audio/wav' });
      const url = URL.createObjectURL(blob);
      ttsCache.set(textToCache, url);
      return url;
    }
  } catch (err) { console.error("Preload failed", err); }
  return null;
};

const speakEn = (text) => {
  if (!text) return;
  window.speechSynthesis.cancel();
  const utterance = new SpeechSynthesisUtterance(text);
  utterance.lang = 'en-US';
  utterance.rate = 0.9;
  window.speechSynthesis.speak(utterance);
};

const getTypeFromFlow = (num) => {
  if ([2, 5, 8].includes(num)) return 1;
  if (num === 3) return 2;
  if ([4, 6, 9].includes(num)) return 3;
  if ([7, 10].includes(num)) return 4;
  if (num === 11) return 6;
  if (num === 12) return 7;
  if (num === 13) return 8;
  if (num === 14) return 9;
  if (num === 15) return 10;
  return null;
};

const getScriptKey = (typeLabel = "") => {
  if (typeLabel.includes("묘사")) return "묘사";
  if (typeLabel.includes("루틴")) return "루틴";
  if (typeLabel.includes("경험")) return "경험";
  return "묘사";
};

// --- 3. Components ---

const IntroScreen = ({ onEnter }) => {
  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-600 to-blue-400 flex flex-col items-center justify-center text-center p-6 animate-in fade-in duration-1000">
      <div className="mb-12 space-y-4">
        <h1 className="text-white font-black text-5xl md:text-7xl tracking-tighter uppercase leading-none">
          OPIC IH/AL MASTER <br className="hidden md:block" /> 
          <span className="text-blue-100 opacity-80 font-bold">WITH TAEO</span>
        </h1>
        <p className="text-white/80 font-bold text-lg md:text-xl tracking-tight text-center">
          채점관의 귀를 사로잡는 고득점 전략 학습 플랫폼
        </p>
      </div>
      <button
        onClick={onEnter}
        className="bg-gradient-to-r from-white to-slate-100 text-slate-900 px-14 py-6 rounded-full shadow-2xl text-xl font-black tracking-tight hover:scale-105 active:scale-95 transition-all duration-300"
      >
        지금 공부하러가기!
      </button>
      <div className="absolute bottom-10 text-white/40 text-xs font-bold uppercase tracking-[0.3em]">
        PREMIUM OPIC STUDY SUITE V2.5
      </div>
    </div>
  );
};

const ThreeRowVocabGrid = ({ items = [] }) => {
  const list = items.slice(0, 12);
  return (
    <div className="bg-white rounded-[2rem] border border-slate-100 shadow-sm p-7 text-slate-900">
      <div className="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-4">
        {list.map((it, idx) => (
          <div key={idx} className={["rounded-2xl px-5 py-4 border text-left", it.emotion ? "bg-rose-50 border-rose-100" : "bg-slate-50 border-slate-100"].join(" ")}>
            <div className="text-[16px] font-black text-slate-900 tracking-tight leading-tight">{it.w}</div>
            <div className={["text-[13px] font-bold mt-2 leading-tight", it.emotion ? "text-rose-600" : "text-slate-500"].join(" ")}>{it.t}</div>
          </div>
        ))}
      </div>
    </div>
  );
};

// --- 4. Main App ---

const App = () => {
  const [entered, setEntered] = useState(() => {
    if (typeof window !== 'undefined') return localStorage.getItem('opic_master_entered') === 'true';
    return false;
  });
  const [activeTab, setActiveTab] = useState('analysis');
  const [selectedItems, setSelectedItems] = useState([]);
  const [currentTopic, setCurrentTopic] = useState("공원 가기");
  const [activeTypeCard, setActiveTypeCard] = useState(null); 
  const [currentFillerCat, setCurrentFillerCat] = useState('전체 리스트');
  const [isMobileMenuOpen, setIsMobileMenuOpen] = useState(false); 

  // --- MockTest States ---
  const [mockStep, setMockStep] = useState(0); 
  const [isQRevealed, setIsQRevealed] = useState(false);
  const [isAnsRevealed, setIsAnsRevealed] = useState(false);
  const [mockTtsStatus, setMockTtsStatus] = useState('idle'); 
  const [currentMockTopic, setCurrentMockTopic] = useState("");
  const [socialIssueIdx, setSocialIssueIdx] = useState(0);
  const [mockTimerSec, setMockTimerSec] = useState(MOCK_LIMIT_SEC);
  const [isMockTimerRunning, setIsMockTimerRunning] = useState(false);
  const [hasStartedThisStep, setHasStartedThisStep] = useState(false);
  const [hasPlayedOnceThisStep, setHasPlayedOnceThisStep] = useState(false);
  const [introStarted, setIntroStarted] = useState(false);
  const [introTimeLeft, setIntroTimeLeft] = useState(INTRO_LIMIT);
  const [isIntroRunning, setIsIntroRunning] = useState(false);

  // --- OPIcAI States ---
  const [opicAiQ, setOpicAiQ] = useState(null);
  const [opicAiShowKo, setOpicAiShowKo] = useState(false);
  const [opicAiUserKo, setOpicAiUserKo] = useState("");
  const [opicAiOutEn, setOpicAiOutEn] = useState("");
  const [opicAiOutKo, setOpicAiOutKo] = useState("");
  const [opicAiLoading, setOpicAiLoading] = useState(false);
  const [opicAiError, setOpicAiError] = useState("");

  const replayTimerRef = useRef(null);

  // --- TTS Core ---
  const speak = async (textToSpeak, onStart, onEnd) => {
    if (!textToSpeak) return;
    if (onStart) onStart();
    try {
      let audioUrl = ttsCache.get(textToSpeak);
      if (!audioUrl) {
        audioUrl = await fetchAndCacheTTS(textToSpeak);
      }
      if (audioUrl) {
        globalAudio.src = audioUrl;
        globalAudio.onended = () => { if (onEnd) onEnd(); };
        await globalAudio.play();
      } else {
        if (onEnd) onEnd();
      }
    } catch (err) {
      console.error("Speech playback error", err);
      if (onEnd) onEnd();
    }
  };

  // --- Reset App Logic ---
  const handleResetApp = () => {
    localStorage.removeItem('opic_master_entered');
    window.location.reload(); 
  };

  const handleResetMock = () => {
    setMockStep(0); 
    setIsQRevealed(false); 
    setIsAnsRevealed(false); 
    setMockTtsStatus('idle');
    setMockTimerSec(MOCK_LIMIT_SEC); 
    setIsMockTimerRunning(false); 
    setHasStartedThisStep(false);
    setHasPlayedOnceThisStep(false);
    setIntroTimeLeft(INTRO_LIMIT); 
    setIsIntroRunning(false);
    if (replayTimerRef.current) clearTimeout(replayTimerRef.current);
  };

  // --- OPIcAI Logic ---
  const getOpicAiAllowedTopics = () => {
    const keys = selectedItems.map((it) => surveyToOpicTopic[it]).filter(Boolean);
    return Array.from(new Set(keys));
  };

  const pickRandomOpicAiQuestion = () => {
    const allowed = getOpicAiAllowedTopics();
    let pool = opicAiQuestionBank;
    if (allowed.length > 0) {
      pool = opicAiQuestionBank.filter(q => allowed.includes(q.topic) || q.topic === 'roleplay' || q.topic === 'advanced');
    }
    if (!pool || pool.length === 0) pool = opicAiQuestionBank;
    const picked = pool[Math.floor(Math.random() * pool.length)];
    setOpicAiQ(picked);
    setOpicAiShowKo(false);
    setOpicAiOutEn("");
    setOpicAiOutKo("");
    setOpicAiError("");
  };

  const pickFillerSet = (n = 3) => {
    const best = fillers.filter(f => f.isBest).map(f => f.phrase);
    const all = fillers.map(f => f.phrase);
    const pool = Array.from(new Set([...best, ...all]));
    const picked = [];
    while (picked.length < n && pool.length > 0) {
      const idx = Math.floor(Math.random() * pool.length);
      picked.push(pool[idx]);
      pool.splice(idx, 1);
    }
    return picked;
  };

  const generateOpicAnswer = async () => {
    setOpicAiError("");
    setOpicAiOutEn("");
    setOpicAiOutKo("");

    const q = opicAiQ?.en?.trim();
    const userKo = opicAiUserKo.trim();

    if (!q) { setOpicAiError("질문이 없습니다. ‘새 질문’을 먼저 눌러주세요."); return; }
    if (!userKo) { setOpicAiError("한국어 답변이 비어있습니다."); return; }

    const fillerSet = pickFillerSet(3);
    const prompt = `
You are an OPIc IH/AL speaking coach. Target difficulty: OPIc 5-5.

[Question]
${q}

[User's Korean intent/content]
${userKo}

[Strict requirements]
- Write ONE natural paragraph in English suitable for a 40-second spoken answer (about 70–95 words).
- Use 2–3 filler expressions naturally (not forced, no repetition) from this list ONLY:
${fillerSet.map((p, i) => `${i + 1}) ${p}`).join("\n")}
- Use correct tense for the question. Add at least one present perfect sentence if it fits naturally.
- Use prepositions naturally (e.g., in, on, at, for, during, across, between).
- Do NOT write bullet points. Do NOT write headings.
- Provide a natural Korean translation of the generated English answer.

[Output format: JSON only]
{
  "en": "English answer (one paragraph)",
  "ko": "Korean translation (one paragraph)"
}
    `.trim();

    setOpicAiLoading(true);
    let attempts = 0;
    const maxAttempts = 5;
    const runApiCall = async () => {
      try {
        const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            contents: [{ parts: [{ text: prompt }] }],
            generationConfig: {
              responseMimeType: "application/json",
              responseSchema: {
                type: "OBJECT",
                properties: {
                  en: { type: "STRING" },
                  ko: { type: "STRING" }
                },
                required: ["en", "ko"]
              }
            }
          })
        });

        if (!response.ok) throw new Error("API call failed");

        const result = await response.json();
        const textContent = result.candidates?.[0]?.content?.parts?.[0]?.text;
        if (textContent) {
          const parsed = JSON.parse(textContent);
          setOpicAiOutEn(parsed.en);
          setOpicAiOutKo(parsed.ko);
          setOpicAiLoading(false);
        } else {
          throw new Error("Empty response");
        }
      } catch (err) {
        if (attempts < maxAttempts) {
          attempts++;
          const delay = Math.pow(2, attempts - 1) * 1000;
          setTimeout(runApiCall, delay);
        } else {
          setOpicAiError("생성 중 오류가 발생했습니다. 잠시 후 다시 시도해 주세요.");
          setOpicAiLoading(false);
        }
      }
    };
    await runApiCall();
  };

  const handleCopyText = (text) => {
    const textArea = document.createElement("textarea");
    textArea.value = text;
    document.body.appendChild(textArea);
    textArea.select();
    try {
      document.execCommand('copy');
    } catch (err) {
      console.error('Copy failed', err);
    }
    document.body.removeChild(textArea);
  };

  const getRandomTopicFromSurvey = () => {
    const mapping = { "공원가기": "공원 가기", "조깅": "조깅/걷기", "걷기": "조깅/걷기", "음악 감상하기": "음악 감상", "국내 여행": "여행 (국내/해외)", "해외 여행": "여행 (국내/해외)", "집에서 보내는 휴가": "집 휴가" };
    const available = selectedItems.map(i => mapping[i]).filter(t => t && masterData[t]);
    if (available.length > 0) {
      let picked = available[Math.floor(Math.random() * available.length)];
      if (picked === currentMockTopic && available.length > 1) picked = available.filter(t => t !== picked)[0];
      return picked;
    }
    return topics[Math.floor(Math.random() * topics.length)];
  };

  const getMockQuestionInfo = () => {
    if (mockStep === 0) return { q: "Please tell me a little bit about yourself.", typeName: "자기소개", topic: "개인 배경", isSocial: false };
    if (mockStep >= 13) {
      const issue = socialIssueBank[socialIssueIdx];
      return { q: issue.q, typeName: "유형 10 · 사회적 이슈", topic: issue.topic, isSocial: true };
    }
    const currentTopicForMock = currentMockTopic || "공원 가기";
    const data = masterData[currentTopicForMock];
    const typeId = getTypeFromFlow(mockStep + 1);
    let qIdx = (typeId === 1) ? 0 : (typeId === 2) ? 1 : 2;
    const typeNames = { 1: "유형 1 · 장소/사물 묘사", 2: "유형 2 · 활동/루틴", 3: "유형 3 · 최근 경험", 4: "유형 4 · 인상적인 경험", 6: "유형 6 · RP 정보 요청", 7: "유형 7 · RP 문제 해결", 8: "유형 8 · 과거 해결 경험", 9: "유형 9 · 비교 및 대조", 10: "유형 10 · 사회적 이슈" };
    return { q: data.expected[qIdx] || data.expected[0], typeName: typeNames[typeId] || "오픽 공통 유형", topic: currentTopicForMock, qIdx: qIdx, isSocial: false };
  };

  const handleMockPlayClick = (questionText) => {
    const shouldStartTimerOnReplay =
      mockStep >= 1 &&
      hasPlayedOnceThisStep &&          
      !hasStartedThisStep &&            
      (mockTtsStatus === 'replay');     

    if (shouldStartTimerOnReplay) {
      setMockTimerSec(MOCK_LIMIT_SEC);     
      setIsMockTimerRunning(true);
      setHasStartedThisStep(true);
    }

    if (mockStep >= 13 && mockTtsStatus === 'idle') {
      setSocialIssueIdx(prev => (prev + 1) % socialIssueBank.length);
    }

    speak(
      questionText,
      () => setMockTtsStatus('loading'),
      () => {
        setHasPlayedOnceThisStep(true);
        setMockTtsStatus('replay');
      }
    );
  };

  const handlePrevStep = () => {
    if (mockStep > 0) {
      setMockStep(mockStep - 1); 
      setIsQRevealed(false); 
      setIsAnsRevealed(false); 
      setMockTtsStatus('idle');
      setMockTimerSec(MOCK_LIMIT_SEC); 
      setIsMockTimerRunning(false); 
      setHasStartedThisStep(false);
      setHasPlayedOnceThisStep(false); 
      if (replayTimerRef.current) clearTimeout(replayTimerRef.current);
    }
  };

  const handleEnterApp = () => {
    setEntered(true);
    localStorage.setItem('opic_master_entered', 'true');
  };

  // --- Effects ---
  useEffect(() => {
    if (!entered) return;
    let targetsArray = [];
    if (activeTab === 'fillers') {
      const filtered = fillers.filter(f => currentFillerCat === '전체 리스트' || f.cat === currentFillerCat);
      targetsArray = filtered.slice(0, 9).map(f => f.phrase);
    } else if (activeTab === 'masterAnswers') {
      const data = masterData[currentTopic];
      if (data) targetsArray = [...data.expected];
    } else if (activeTab === 'mocktest') {
      const info = getMockQuestionInfo();
      if (info && info.q) targetsArray = [info.q];
    }
    
    const runQueue = async () => {
      const filteredTargets = targetsArray.filter(t => t && (typeof t === 'string') && !ttsCache.has(t));
      for (const textToCache of filteredTargets) {
        await fetchAndCacheTTS(textToCache);
      }
    };
    runQueue();
  }, [activeTab, currentTopic, currentFillerCat, mockStep, entered]);

  useEffect(() => {
    if (activeTab !== 'mocktest') return;
    if (!isMockTimerRunning) return;
    const t = setInterval(() => {
      setMockTimerSec(prev => {
        if (prev <= 1) {
          clearInterval(t);
          setIsMockTimerRunning(false);
          return 0;
        }
        return prev - 1;
      });
    }, 1000);
    return () => clearInterval(t);
  }, [activeTab, isMockTimerRunning]);

  useEffect(() => {
    if (!isIntroRunning) return;
    if (introTimeLeft <= 0) {
      setIsIntroRunning(false);
      return;
    }
    const t = setInterval(() => {
      setIntroTimeLeft(prev => prev - 1);
    }, 1000);
    return () => clearInterval(t);
  }, [isIntroRunning, introTimeLeft]);

  useEffect(() => {
    if (activeTab === 'mocktest') {
      const isTopicSetRange = mockStep >= 1 && mockStep <= 12;
      const isSetStart = (mockStep === 1) || ((mockStep - 1) % 3 === 0);

      if (isTopicSetRange && isSetStart) {
        setCurrentMockTopic(getRandomTopicFromSurvey());
      }

      setMockTimerSec(MOCK_LIMIT_SEC);
      setIsMockTimerRunning(false);
      setHasStartedThisStep(false);
      setHasPlayedOnceThisStep(false); 

      if (mockStep === 0) {
        setIntroStarted(false);
        setIsIntroRunning(false);
        setIntroTimeLeft(INTRO_LIMIT);
      }
      setMockTtsStatus('idle');
      if (replayTimerRef.current) clearTimeout(replayTimerRef.current);
    }
  }, [mockStep, activeTab]);

  useEffect(() => {
    if (activeTab === 'opicAI' && !opicAiQ) {
      pickRandomOpicAiQuestion();
    }
  }, [activeTab]);

  useEffect(() => {
    if (mockTtsStatus === 'replay') replayTimerRef.current = setTimeout(() => setMockTtsStatus('expired'), 4000);
    return () => { if (replayTimerRef.current) clearTimeout(replayTimerRef.current); };
  }, [mockTtsStatus]);

  // --- Sub-Renderers ---

  const renderAnalysis = () => (
    <div className="max-w-6xl mx-auto space-y-12 animate-in fade-in slide-in-from-bottom-8 duration-700 py-10 px-6 text-slate-900 text-center">
      <div className="bg-white p-10 rounded-[3rem] shadow-xl border border-blue-50 relative overflow-hidden text-left">
        <h2 className="text-3xl font-black mb-12 tracking-tighter text-center">OPIc 5-5 난이도 시험 구성 분석</h2>
        <div className="flex flex-col lg:flex-row gap-12 items-start">
          <div className="w-full lg:w-[400px] space-y-2.5">
            <h3 className="text-sm font-black text-blue-400 tracking-wide mb-6 px-2 text-center">Full question flow</h3>
            {comboFlowData.map((item, idx) => (
              <div key={idx} className="flex items-center gap-4 group cursor-pointer" onClick={() => setActiveTypeCard(getTypeFromFlow(item.num))}>
                <div className={`w-10 h-10 rounded-xl flex items-center justify-center font-black flex-shrink-0 border-2 transition-all group-hover:bg-blue-600 group-hover:text-white group-hover:border-blue-600 group-hover:scale-105 ${item.textColor || 'text-blue-900'} ${item.color} ${item.border}`}>{item.num}</div>
                <div className={`flex-1 p-3 rounded-2xl border-2 font-black text-sm transition-all group-hover:bg-blue-600 group-hover:text-white group-hover:border-blue-600 group-hover:translate-x-1 ${item.color} ${item.border} ${item.textColor || 'text-blue-900'} shadow-sm tracking-tight`}>{item.label}</div>
              </div>
            ))}
          </div>
          <div className="flex-1 sticky top-24">
            <h3 className="text-xl font-black border-b-4 border-blue-600 pb-3 mb-8 tracking-tight flex items-center gap-3"><i className="fas fa-list-check text-blue-600"></i> 필수 공략 유형 10선</h3>
            <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
              {[
                { id: 1, title: "현재 시제 묘사", content: "장소, 사물, 사람의 특징을 생생하게 설명", star: false },
                { id: 2, title: "활동 및 루틴", content: "시간 순서에 따른 습관이나 단계별 과정", star: false },
                { id: 3, title: "최근/과거 경험", content: "과거 시제 사용 필수, 가장 중요한 득점 포인트", star: true },
                { id: 4, title: "인상적인 경험", content: "감정 표현과 특별한 계기 중심의 서사", star: false },
                { id: 6, title: "RP - 정보 요청", content: "3-4가지 구체적인 질문 던지기", star: false },
                { id: 7, title: "RP - 상황 해결", content: "문제 상황 설명 후 2-3가지 대안 제시", star: true },
                { id: 8, title: "과거 문제 해결", content: "유형 7과 연계된 실제 나의 경험담", star: false },
                { id: 9, title: "비교 및 대조", content: "과거 vs 현재, A vs B의 차이점 분석", star: false },
                { id: 10, title: "사회적 이슈", content: "주제와 관련된 최근 트렌드와 나의 생각", star: true },
              ].map((type) => (
                <div key={type.id} className={`p-6 rounded-3xl border-2 transition-all duration-300 shadow-sm ${activeTypeCard === type.id ? 'bg-blue-600 text-white border-blue-600 shadow-lg' : 'bg-white border-slate-100 text-slate-800'}`}>
                  <div className="flex justify-between items-center mb-2"><span className={`text-[10px] font-black tracking-wider ${activeTypeCard === type.id ? 'text-blue-100' : 'text-blue-500'}`}>Type {type.id}</span>{type.star && (<i className="fas fa-star text-yellow-400"></i>)}</div>
                  <h4 className="text-lg font-black mb-1">{type.title}</h4>
                  <p className="text-sm font-medium leading-relaxed opacity-80">{type.content}</p>
                </div>
              ))}
            </div>
          </div>
        </div>
      </div>
    </div>
  );

  const renderSurvey = () => (
    <div className="max-w-5xl mx-auto py-12 px-6 animate-in fade-in duration-500 text-slate-900 text-center">
      <div className="bg-white p-12 rounded-[3.5rem] shadow-2xl border border-blue-50 space-y-12">
        <h2 className="text-4xl font-black tracking-tighter">Strategic survey setup</h2>
        <p className="text-slate-500 font-bold text-center">오픽 고득점을 위한 <span className="text-blue-600 underline underline-offset-4">필수 12가지</span> 전략적 선택</p>
        <div className="inline-flex items-center gap-6 bg-blue-50 px-10 py-4 rounded-[2rem] border border-blue-100 shadow-inner">
          <span className="text-sm font-black text-blue-400 tracking-wide leading-none">Selection progress</span>
          <span className={`text-3xl font-black leading-none ${selectedItems.length === 12 ? 'text-blue-600' : 'text-slate-400'}`}>{selectedItems.length} / 12</span>
        </div>
        <div className="grid grid-cols-1 gap-12 text-left">
          {surveyCategories.map((cat, idx) => (
            <div key={idx} className="space-y-6">
              <h3 className="text-xl font-black text-blue-900 flex items-center gap-4 text-left">
                <span className="w-10 h-10 rounded-2xl bg-blue-950 text-white flex items-center justify-center text-sm shadow-lg font-bold">{idx + 1}</span>{cat.title}
              </h3>
              <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
                {cat.items.map(item => {
                  const isChecked = selectedItems.includes(item);
                  return (
                    <label key={item} className={`relative flex items-center gap-3 p-5 border-2 rounded-[1.5rem] cursor-pointer transition-all ${isChecked ? 'bg-blue-600 border-blue-600 text-white shadow-xl scale-[1.02]' : 'bg-white border-slate-50 text-slate-600 hover:border-blue-200'}`}>
                      <input type="checkbox" className="hidden" checked={isChecked} onChange={() => setSelectedItems(prev => prev.includes(item) ? prev.filter(i => i !== item) : prev.length < 12 ? [...prev, item] : prev)} />
                      <i className={`fas ${isChecked ? 'fa-check-circle' : 'fa-circle-plus opacity-30'} text-xl`}></i>
                      <span className="text-[14px] font-black tracking-tight">{item}</span>
                      {cat.strategy.includes(item) && !isChecked && <span className="absolute -top-1 -right-1 bg-blue-600 text-[8px] font-black px-2 py-1 rounded-full text-white shadow-sm animate-bounce">Pick</span>}
                    </label>
                  );
                })}
              </div>
            </div>
          ))}
        </div>
      </div>
    </div>
  );

  const renderMasterAnswers = () => {
    const data = masterData[currentTopic];
    return (
      <div className="max-w-7xl mx-auto py-10 px-6 animate-in fade-in duration-700 space-y-10 text-slate-900 text-center">
        <h2 className="text-4xl font-black tracking-tighter leading-none text-center">Master <span className="text-blue-600">answers</span> lab</h2>
        <div className="flex items-center justify-center gap-3 overflow-x-auto pb-4 no-scrollbar">
          {topics.map(name => (
            <button key={name} onClick={() => setCurrentTopic(name)} className={`flex-shrink-0 px-8 py-3 rounded-full border-2 font-black text-sm transition-all shadow-sm whitespace-nowrap ${currentTopic === name ? 'bg-blue-600 text-white border-blue-600 shadow-blue-200 scale-105' : 'bg-white text-slate-400 border-slate-100 hover:border-blue-200 hover:text-slate-600'}`}>{name}</button>
          ))}
        </div>
        <div className="bg-white p-14 rounded-[4rem] shadow-2xl border border-blue-50 space-y-20 relative w-full text-left font-bold">
            <h2 className="text-5xl font-black text-blue-950 tracking-tighter leading-none text-left">{currentTopic} <span className="text-blue-600">pack</span></h2>
            <div className="space-y-5">
                {data.expected.map((q, i) => (
                    <div key={i} className="p-10 bg-blue-50/40 rounded-[2.5rem] text-xl font-black text-blue-900 border-2 border-blue-50/50 shadow-sm leading-relaxed relative group transition-all hover:bg-white hover:shadow-xl text-left font-bold">
                        <button onClick={() => speak(q)} className="absolute top-8 right-10 text-blue-200 group-hover:text-blue-600 transition-colors font-bold"><i className="fas fa-volume-up text-xl"></i></button>
                        "{q}"
                    </div>
                ))}
            </div>
            <div className="space-y-24">
                {data.scripts.map((s, i) => {
                    const key = getScriptKey(s.type);
                    const vocab = vocabMap[currentTopic]?.[key] || [];
                    return (
                        <div key={i} className="relative group text-left">
                            <h5 className="text-2xl font-black text-blue-950 mb-6 tracking-tight text-left">{s.type}</h5>
                            <div className="mb-10 text-left"><p className="text-[13px] font-black text-slate-400 tracking-tight mb-4 text-left uppercase">암기할 단어</p><ThreeRowVocabGrid items={vocab} /></div>
                            <div className="p-14 bg-blue-950 text-blue-50 rounded-[4rem] shadow-2xl relative overflow-hidden border-b-[20px] border-blue-900 text-left font-bold">
                                <button onClick={() => speak(s.text)} className="absolute top-10 right-12 text-blue-400/60 hover:text-blue-200 z-10 transition-colors font-bold"><i className="fas fa-volume-up text-3xl"></i></button>
                                <p className="text-3xl leading-relaxed font-medium tracking-tighter whitespace-pre-line z-10 relative font-bold">
                                  {s.text.split('**').map((part, pIdx) => pIdx % 2 === 1 ? <span key={pIdx} className="text-blue-400 font-black underline underline-offset-[14px] decoration-blue-700/50 font-bold">{part}</span> : part)}
                                </p>
                            </div>
                            <div className="mt-10 p-12 bg-slate-50 text-blue-950/50 rounded-[3rem] text-2xl font-black border-2 border-slate-100 shadow-inner leading-relaxed text-center transition-all group-hover:text-blue-900 group-hover:bg-white font-bold">{s.trans}</div>
                        </div>
                    );
                })}
            </div>
        </div>
      </div>
    );
  };

  const renderFillers = () => {
    const categories = ["전체 리스트", "질문 리액션", "시간 벌기", "감정 및 강조", "설명 및 정정", "전환 및 추가", "깔끔한 마무리"];
    const filteredFillers = fillers.filter(f => currentFillerCat === '전체 리스트' || f.cat === currentFillerCat);
    return (
      <div className="max-w-7xl mx-auto py-10 px-6 animate-in fade-in duration-700 space-y-10 text-slate-900 text-center font-bold">
        <h2 className="text-4xl font-black tracking-tighter text-center leading-none">OPIc filler vault <span className="text-blue-600">100</span></h2>
        <div className="flex items-center justify-center gap-3 overflow-x-auto pb-4 no-scrollbar font-bold">
          {categories.map(name => {
            const count = fillers.filter(f => name === '전체 리스트' ? true : f.cat === name).length;
            const isActive = currentFillerCat === name;
            return (
              <button key={name} onClick={() => setCurrentFillerCat(name)}
                className={`flex-shrink-0 px-6 py-3 rounded-full border shadow-sm transition-all text-[13px] font-semibold flex items-center gap-3 ${isActive ? 'bg-blue-600 border-blue-600 text-white' : 'bg-white border-slate-100 text-slate-500 hover:border-slate-300'}`}
              >
                {name} <span className={`text-[10px] font-bold ${isActive ? 'text-blue-100' : 'text-slate-300'}`}>{count}</span>
              </button>
            );
          })}
        </div>
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
          {filteredFillers.map((f, idx) => (
            <div key={idx} onClick={() => speak(f.phrase)} className="bg-white rounded-[28px] border border-slate-100 shadow-[0_8px_30px_rgba(15,23,42,0.06)] p-10 pt-12 relative group transition-all hover:-translate-y-1 hover:shadow-[0_14px_40px_rgba(15,23,42,0.08)] cursor-pointer text-left font-bold">
              {f.isBest && (<span className="absolute top-8 left-8 text-[11px] font-bold px-4 py-1.5 rounded-full bg-blue-600 text-white shadow-md uppercase font-bold">Best</span>)}
              <span className="absolute top-[68px] left-8 text-[12px] font-semibold px-4 py-2 rounded-full bg-slate-100 text-slate-500 font-bold">{f.cat}</span>
              <button onClick={(e) => { e.stopPropagation(); speak(f.phrase); }} className="absolute top-8 right-8 w-10 h-10 rounded-full flex items-center justify-center text-slate-300 hover:text-blue-600 hover:bg-blue-50 transition shadow-sm border border-slate-50"><i className="fas fa-volume-up font-bold"></i></button>
              <h3 className="mt-24 text-[30px] leading-[1.15] font-extrabold text-slate-900 tracking-tighter text-left">{f.phrase}</h3>
              <p className="mt-4 text-[20px] font-extrabold text-blue-600 tracking-tight text-left font-bold">{f.meaning}</p>
              <div className="mt-10 bg-slate-50 rounded-[22px] p-7 border border-slate-100 shadow-inner text-left"><p className="text-slate-500 text-[16px] leading-relaxed font-bold"> <span className="text-blue-400 font-bold mr-2 uppercase">Ex:</span>"{f.ex}"</p></div>
            </div>
          ))}
        </div>
      </div>
    );
  };

  const renderMockTest = () => {
    const info = getMockQuestionInfo();
    const isIntro = mockStep === 0;
    const mm = String(Math.floor(mockTimerSec / 60)).padStart(2, '0');
    const ss = String(mockTimerSec % 60).padStart(2, '0');

    return (
      <div className="max-w-5xl mx-auto py-10 md:py-20 px-6 animate-in fade-in zoom-in duration-500 text-center text-slate-900 font-bold">
          <div className="mb-12 flex flex-col md:flex-row items-center justify-between gap-6 text-left">
            <div className="text-left space-y-1">
                <h3 className="text-3xl font-black tracking-tighter text-slate-900">OPIc actual test · <span className="text-blue-600">{mockStep + 1}번 문제</span></h3>
                <p className="text-slate-400 font-bold tracking-wide text-xs uppercase leading-none">Step {mockStep + 1} of 15</p>
            </div>
            <div className="flex items-center gap-2 flex-nowrap overflow-x-auto no-scrollbar max-w-full pb-2">
                {[...Array(15)].map((_, i) => (
                  <div key={i} className={`w-7 h-7 rounded-full flex items-center justify-center text-[10px] font-black flex-shrink-0 transition-all duration-300 shadow-sm ${mockStep > i ? 'bg-blue-600 text-white' : mockStep === i ? 'border-2 border-blue-600 bg-white text-blue-600 scale-105 shadow-md ring-2 ring-blue-50 font-bold' : 'bg-slate-200 text-slate-400'}`}>{i + 1}</div>
                ))}
            </div>
          </div>
          
          <div className="bg-white rounded-[4rem] shadow-2xl border border-slate-100 p-8 md:p-16 flex flex-col items-center space-y-12 relative min-h-[550px] h-auto text-center font-bold">
            <div className="absolute top-0 left-0 w-full h-2.5 bg-slate-50"><div className="h-full bg-blue-600 transition-all duration-1000 ease-out" style={{ width: `${((mockStep + 1) / 15) * 100}%` }}></div></div>
            
            {isIntro ? (
              <div className="animate-in fade-in slide-in-from-bottom-4 space-y-8 flex flex-col items-center w-full">
                <div className="flex items-center gap-3 text-slate-900 font-bold">
                  <span className={`w-2.5 h-2.5 rounded-full ${isIntroRunning ? 'bg-red-500 animate-pulse' : 'bg-red-300'}`}></span>
                  <span className="text-xs font-black tracking-wide text-slate-500 uppercase">Recording</span>
                  <span className="font-mono font-black tabular-nums text-2xl">
                    00:{String(Math.max(introTimeLeft, 0)).padStart(2, '0')}
                  </span>
                </div>
                <div className="space-y-4 text-center font-bold"><h4 className="text-4xl font-black text-slate-900 tracking-tighter leading-tight font-bold">"Let's start the interview now."</h4><p className="text-xl text-slate-400 font-bold leading-relaxed max-w-md mx-auto text-center font-bold">간단하게 자기소개를 해주세요. <br/>이름, 가족, 취미 등을 자유롭게 이야기하시면 됩니다.</p></div>
                <div className="flex items-center gap-4 pt-4"><button onClick={() => { setIntroStarted(true); setIntroTimeLeft(INTRO_LIMIT); setIsIntroRunning(true); }} className="px-10 py-5 rounded-[1.5rem] font-black text-sm shadow-xl transition-all bg-blue-600 text-white hover:bg-blue-700 active:scale-95">{isIntroRunning ? "진행 중" : (introStarted ? "다시 시작" : "시험 시작")}</button><button onClick={() => { setIsIntroRunning(false); setMockStep(1); }} className="px-12 py-5 rounded-[1.5rem] font-black text-sm shadow-xl transition-all bg-blue-600 text-white hover:bg-blue-700 active:scale-95 flex items-center gap-3 font-bold">다음으로 <i className="fas fa-arrow-right"></i></button></div>
              </div>
            ) : (
              <div className="flex flex-col items-center w-full space-y-12">
                <div className="flex items-center justify-center gap-3 text-slate-400 font-black tracking-wide uppercase"><span className={`w-2.5 h-2.5 rounded-full ${isMockTimerRunning ? 'bg-red-500 animate-pulse' : 'bg-rose-300'}`}></span><span className="text-[12px]">Recording</span><span className="text-slate-900 tracking-normal text-2xl font-black tabular-nums">{mm}:{ss}</span></div>
                <div className="flex flex-col items-center justify-center w-full space-y-12 text-center">
                  <div className="relative w-36 h-36 flex items-center justify-center">{mockTtsStatus === 'loading' && (<div className="absolute inset-0 rounded-full animate-spin z-0" style={{ background: 'conic-gradient(from 0deg, #2563eb, #93c5fd, transparent)', padding: '4px', mask: 'radial-gradient(farthest-side, transparent calc(100% - 6px), black calc(100% - 5px))', WebkitMask: 'radial-gradient(farthest-side, transparent calc(100% - 6px), black calc(100% - 5px))' }}></div>)}<button onClick={() => handleMockPlayClick(info.q)} disabled={mockTtsStatus === 'loading' || mockTtsStatus === 'playing' || mockTtsStatus === 'expired'} className={`w-28 h-28 rounded-full flex items-center justify-center shadow-2xl transition-all relative z-10 ${mockTtsStatus === 'playing' ? 'bg-slate-500 cursor-not-allowed shadow-none' : ''} ${(mockTtsStatus === 'idle' || mockTtsStatus === 'replay') ? 'bg-blue-600 text-white hover:scale-110 active:scale-95 hover:shadow-blue-200' : ''} ${mockTtsStatus === 'loading' ? 'bg-white text-blue-600 cursor-not-allowed font-bold' : ''} ${mockTtsStatus === 'expired' ? 'bg-slate-100 text-slate-300 cursor-not-allowed shadow-none border-2 border-slate-50 font-bold' : ''}`}>{mockTtsStatus === 'idle' && <i className="fas fa-play text-3xl ml-1"></i>}{mockTtsStatus === 'loading' && <span className="sr-only">Loading...</span>}{mockTtsStatus === 'playing' && <i className="fas fa-play text-3xl ml-1 opacity-50"></i>}{(mockTtsStatus === 'replay' || mockTtsStatus === 'expired') && <i className="fas fa-rotate-right text-3xl"></i>}</button>{mockTtsStatus === 'playing' && <div className="absolute -bottom-8 whitespace-nowrap text-[10px] font-black text-slate-400 tracking-wide animate-pulse leading-none">Now playing...</div>}{mockTtsStatus === 'replay' && <div className="absolute -bottom-8 whitespace-nowrap text-[10px] font-black text-blue-400 tracking-wide animate-bounce leading-none font-bold">Replay (4s)</div>}</div>
                  <div className="text-center min-h-[140px] flex flex-col items-center justify-center max-w-xl text-center font-bold font-bold">{!isQRevealed ? <button onClick={() => setIsQRevealed(true)} className="text-2xl font-black text-slate-300 hover:text-blue-500 transition-colors border-b-4 border-slate-50 pb-3 decoration-dotted font-bold">질문 텍스트 보기</button> : <div className="animate-in fade-in slide-in-from-top-4 space-y-6 text-center"><div className="inline-flex items-center gap-3 bg-blue-50 px-5 py-2 rounded-xl"><span className="text-[11px] font-black text-blue-600 tracking-wide leading-none">{info.typeName}</span><span className="text-slate-200">|</span><span className="text-[11px] font-black text-slate-400 leading-none">{info.topic}</span></div><p className="text-3xl font-black leading-snug tracking-tighter text-slate-900 font-bold">"{info.q}"</p></div>}</div>
                </div>
              </div>
            )}
            {!isIntro && (<div className="flex gap-6"><button onClick={() => setIsAnsRevealed(!isAnsRevealed)} className="px-12 py-5 bg-slate-900 text-white rounded-[1.5rem] font-black text-sm hover:bg-black transition-all shadow-xl font-bold">스크립트 확인</button><button onClick={() => { setIsMockTimerRunning(false); setHasStartedThisStep(false); setMockTimerSec(MOCK_LIMIT_SEC); setIsQRevealed(false); setIsAnsRevealed(false); setMockStep(p => (p < 14 ? p + 1 : 0)); }} className="px-12 py-5 bg-blue-600 text-white rounded-[1.5rem] font-black text-sm hover:bg-blue-700 transition-all shadow-xl flex items-center gap-4 font-bold">{mockStep < 14 ? "다음으로" : "처음으로"} <i className="fas fa-arrow-right"></i></button></div>)}
          </div>
          <div className="mt-8 flex justify-center gap-4"><button onClick={handleResetMock} className="flex items-center gap-2 px-6 py-3 bg-white text-slate-500 border border-slate-200 rounded-2xl font-bold text-xs hover:bg-slate-50 hover:text-slate-700 transition-all shadow-sm leading-none font-bold uppercase"><i className="fas fa-undo-alt text-[10px]"></i> 처음으로</button><button onClick={handlePrevStep} disabled={mockStep === 0} className={`flex items-center gap-2 px-6 py-3 border rounded-2xl font-bold text-xs transition-all shadow-sm leading-none font-bold uppercase ${mockStep === 0 ? 'bg-slate-50 text-slate-300 border-slate-100 cursor-not-allowed' : 'bg-white text-slate-600 border-slate-200 hover:bg-slate-50'}`}><i className="fas fa-chevron-left text-[10px]"></i> 이전으로</button></div>
          {isAnsRevealed && !isIntro && (<div className="w-full mt-10 animate-in slide-in-from-top-8 duration-500 text-left font-bold font-bold"><div className="bg-blue-50/50 p-14 rounded-[3.5rem] border-2 border-blue-100 relative group text-left shadow-inner font-bold"><button onClick={() => speak(info.isSocial ? info.q : masterData[info.topic].scripts[info.qIdx].text)} className="absolute top-10 right-12 text-blue-300 hover:text-blue-600 transition-colors font-bold"><i className="fas fa-volume-up text-2xl"></i></button><div className="flex items-center gap-4 mb-8 font-bold"><span className="bg-blue-600 text-white px-4 py-1.5 rounded-xl text-[10px] font-black tracking-wide leading-none font-bold">Master answer</span><h4 className="text-xl font-black text-blue-900 tracking-tight underline decoration-blue-200 underline-offset-8 uppercase text-left font-bold">Study reference</h4></div>{info.isSocial ? <p className="text-3xl font-black leading-relaxed tracking-tight mb-10 text-left text-slate-900 font-bold font-bold">"{info.q}" <br/>(사회적 이슈는 정해진 정답보다 본인의 주관적인 의견 정리가 중요합니다.)</p> : <><p className="text-3xl font-black leading-relaxed tracking-tight mb-10 text-left text-slate-900 font-bold font-bold font-bold">{masterData[info.topic].scripts[info.qIdx].text.split('**').map((part, pIdx) => pIdx % 2 === 1 ? <span key={pIdx} className="text-blue-600 underline decoration-blue-100 underline-offset-8 decoration-4 font-bold">{part}</span> : part)}</p><div className="pt-10 border-t-2 border-blue-100/50 text-left font-bold"><p className="text-xl text-slate-500 font-bold leading-relaxed text-left font-bold">{masterData[info.topic].scripts[info.qIdx].trans}</p></div></>}</div></div>)}
      </div>
    );
  };

  const renderOpicAI = () => {
    const q = opicAiQ;
    return (
      <div className="max-w-5xl mx-auto py-10 px-6 animate-in fade-in duration-700 space-y-8 text-slate-900 font-bold">
        <div className="flex items-end justify-between gap-4">
          <div>
            <h2 className="text-4xl font-black tracking-tighter leading-none font-bold font-bold">
              OPIc <span className="text-blue-600">AI</span>
            </h2>
            <p className="text-slate-400 font-bold mt-2 font-bold">
              서베이 선택 기반 랜덤 질문 → 한국어로 답변 → 원어민 답변 생성
            </p>
          </div>
          <button
            onClick={pickRandomOpicAiQuestion}
            className="px-8 py-4 rounded-[1.5rem] bg-blue-600 text-white font-black shadow-xl hover:bg-blue-700 active:scale-95 transition-all font-bold"
          >
            새 질문
          </button>
        </div>

        <div className="bg-white rounded-[3rem] border border-slate-100 shadow-2xl p-10 relative overflow-hidden font-bold font-bold">
          <div className="flex items-start justify-between gap-4">
            <div className="flex-1 font-bold">
              <div className="text-[12px] font-black tracking-wide text-slate-400 uppercase font-bold">
                Random question · OPIc 5-5
              </div>
              <div className="mt-4 text-3xl font-black tracking-tight leading-snug text-slate-900 font-bold font-bold">
                {q ? `"${q.en}"` : "새 질문을 눌러 질문을 생성하세요."}
              </div>
            </div>

            <div className="flex items-center gap-2 font-bold font-bold">
              <button
                onClick={() => q && speakEn(q.en)}
                disabled={!q}
                className={`w-12 h-12 rounded-full border flex items-center justify-center transition-all shadow-sm font-bold
                  ${q ? "bg-white text-slate-500 hover:text-blue-600 hover:bg-blue-50 border-slate-100" : "bg-slate-50 text-slate-300 border-slate-50 cursor-not-allowed font-bold"}`}
                title="미국식 발음으로 듣기"
              >
                <i className="fas fa-volume-up font-bold"></i>
              </button>

              <button
                onClick={() => setOpicAiShowKo(v => !v)}
                disabled={!q}
                className={`px-5 py-3 rounded-2xl font-black text-sm transition-all border shadow-sm font-bold
                  ${q ? "bg-white text-slate-500 hover:text-slate-800 border-slate-100 font-bold" : "bg-slate-50 text-slate-300 border-slate-50 cursor-not-allowed font-bold"}`}
              >
                {opicAiShowKo ? "해석 닫기" : "해석 보기"}
              </button>
            </div>
          </div>

          {opicAiShowKo && q && (
            <div className="mt-8 bg-slate-50 border border-slate-100 rounded-[2rem] p-8 font-bold">
              <div className="text-[12px] font-black tracking-wide text-slate-400 uppercase font-bold">
                Korean translation
              </div>
              <div className="mt-3 text-xl font-black text-slate-800 leading-relaxed font-bold font-bold">
                {q.ko}
              </div>
            </div>
          )}
        </div>

        <div className="grid grid-cols-1 lg:grid-cols-2 gap-8 font-bold">
          <div className="bg-white rounded-[3rem] border border-slate-100 shadow-2xl p-10 space-y-6 font-bold">
            <div>
              <div className="text-[12px] font-black tracking-wide text-slate-400 uppercase font-bold font-bold">
                한국어 답변 입력
              </div>
              <div className="mt-2 text-xl font-black text-slate-900 font-bold font-bold">
                원하는 내용을 한국어로 써주세요
              </div>
            </div>

            <textarea
              value={opicAiUserKo}
              onChange={(e) => setOpicAiUserKo(e.target.value)}
              placeholder="예) 저는 보통 공원에 주말에 가요. 가서 음악을 들으면서 걷는 걸 좋아해요. 지난 번에는 강아지를 데리고 갔는데 정말 힐링됐어요."
              className="w-full min-h-[220px] rounded-[2rem] border border-slate-100 bg-slate-50 p-6 text-slate-800 font-bold outline-none focus:ring-2 focus:ring-blue-100 font-bold"
            />

            <button
              onClick={generateOpicAnswer}
              disabled={!opicAiQ || !opicAiUserKo.trim() || opicAiLoading}
              className={`w-full px-10 py-5 rounded-[1.8rem] font-black shadow-xl transition-all font-bold
                ${(!opicAiQ || !opicAiUserKo.trim() || opicAiLoading) ? "bg-slate-200 text-slate-400 cursor-not-allowed font-bold" : "bg-blue-600 text-white hover:bg-blue-700 active:scale-95"}`}
            >
              {opicAiLoading ? "답변 생성 중..." : "오픽문장 만들기"}
            </button>
          </div>

          <div className="bg-white rounded-[3rem] border border-slate-100 shadow-2xl p-10 space-y-6 flex flex-col font-bold font-bold font-bold">
            <div className="flex items-center justify-between gap-4 font-bold">
              <div>
                <div className="text-[12px] font-black tracking-wide text-slate-400 uppercase font-bold">
                  OPIc style result
                </div>
                <div className="mt-2 text-xl font-black text-slate-900 font-bold">
                  원어민 답변 + 해석
                </div>
              </div>

              <div className="flex items-center gap-2 font-bold">
                <button
                  onClick={() => opicAiOutEn && speakEn(opicAiOutEn)}
                  disabled={!opicAiOutEn}
                  className={`w-12 h-12 rounded-full border flex items-center justify-center transition-all shadow-sm font-bold
                    ${opicAiOutEn ? "bg-white text-slate-500 hover:text-blue-600 hover:bg-blue-50 border-slate-100" : "bg-slate-50 text-slate-300 border-slate-50 cursor-not-allowed"}`}
                  title="답변 미국식으로 듣기"
                >
                  <i className="fas fa-volume-up font-bold"></i>
                </button>
                <button
                  onClick={() => { setOpicAiOutEn(""); setOpicAiOutKo(""); setOpicAiError(""); }}
                  className="px-6 py-3 rounded-2xl font-black text-sm border shadow-sm transition-all bg-white text-slate-600 hover:bg-slate-50 border-slate-100 font-bold"
                >
                  지우기
                </button>
              </div>
            </div>

            {opicAiError && (
              <div className="bg-rose-50 border border-rose-100 rounded-[2rem] p-6 text-rose-700 font-bold text-sm">
                {opicAiError}
              </div>
            )}

            <div className="bg-slate-950 text-slate-100 rounded-[2.5rem] p-8 border border-slate-900 flex-1 min-h-[360px] font-bold">
              {opicAiLoading ? (
                <div className="h-full flex flex-col items-center justify-center space-y-4 font-bold">
                  <div className="w-10 h-10 border-4 border-blue-600 border-t-transparent rounded-full animate-spin font-bold"></div>
                  <div className="text-slate-400 font-black tracking-wide uppercase animate-pulse font-bold">Generating...</div>
                </div>
              ) : opicAiOutEn ? (
                <div className="space-y-8 animate-in fade-in duration-500 font-bold">
                  <div>
                    <div className="flex items-center justify-between font-bold">
                      <div className="text-[11px] font-black tracking-wide uppercase text-blue-400 font-bold">English Answer</div>
                      <button onClick={() => handleCopyText(opicAiOutEn)} className="text-[10px] text-slate-500 hover:text-blue-400 uppercase font-black tracking-wide font-bold">Copy</button>
                    </div>
                    <p className="mt-4 text-[17px] leading-relaxed font-bold text-slate-100 whitespace-pre-wrap">
                      {opicAiOutEn}
                    </p>
                  </div>

                  <div className="pt-8 border-t border-slate-800 font-bold">
                    <div className="text-[11px] font-black tracking-wide uppercase text-slate-500 font-bold">Korean Translation</div>
                    <p className="mt-4 text-[15px] leading-relaxed font-bold text-slate-400 whitespace-pre-wrap font-bold">
                      {opicAiOutKo}
                    </p>
                  </div>
                </div>
              ) : (
                <div className="h-full flex items-center justify-center text-center font-bold">
                  <p className="text-slate-600 font-bold leading-relaxed max-w-[200px] font-bold">
                    왼쪽에 내용을 입력하고 버튼을 누르면 AI가 오픽용 답변을 만들어 드립니다.
                  </p>
                </div>
              )}
            </div>

            <div className="text-slate-400 text-[12px] font-bold leading-relaxed px-4">
              5-5 레벨에서는 자연스러운 필러 사용과 시제 관리가 생명입니다. 위 답변을 소리 내어 읽어보세요!
            </div>
          </div>
        </div>
      </div>
    );
  };

  if (!entered) return <IntroScreen onEnter={handleEnterApp} />;

  return (
    <div className="min-h-screen flex flex-col bg-[#FDFDFD] font-sans antialiased overflow-x-hidden text-slate-900 font-bold">
      <header className="sticky top-0 z-50 bg-white/80 backdrop-blur-xl border-b border-slate-100 px-6 sm:px-10 font-bold">
        <div className="max-w-7xl mx-auto flex items-center justify-between h-20 font-bold">
          <div className="flex items-center cursor-pointer group font-bold" onClick={handleResetApp}>
            <h1 className="text-[28px] font-bold tracking-tighter uppercase leading-none group-hover:scale-95 transition-transform font-bold">
               <span className="text-slate-900 font-bold">OPIc </span>
               <span className="text-blue-600 font-bold">IH</span>
               <span className="text-slate-900 font-bold"> MASTER</span>
            </h1>
          </div>
          <nav className="hidden lg:flex items-center justify-center gap-10 h-full whitespace-nowrap font-bold">
            {[
              {id: 'analysis', label: '유형 분석'}, 
              {id: 'survey', label: '서베이 설정'}, 
              {id: 'masterAnswers', label: '만능 스크립트'}, 
              {id: 'fillers', label: '필러 표현'}, 
              {id: 'mocktest', label: '실전 모의고사'},
              {id: 'opicAI', label: '오픽AI'}
            ].map(tab => (
              <button key={tab.id} onClick={() => setActiveTab(tab.id)} className={`h-full px-4 border-b-[4px] transition-all font-bold text-sm tracking-tight ${activeTab === tab.id ? 'border-blue-600 text-blue-600' : 'border-transparent text-slate-400 hover:text-slate-800'}`}>{tab.label}</button>
            ))}
          </nav>
          <div className="lg:hidden flex items-center font-bold">
            <button onClick={() => setIsMobileMenuOpen(true)} className="p-2 text-slate-600 hover:text-blue-600 transition-colors font-bold">
              <i className="fas fa-bars text-2xl font-bold font-bold font-bold"></i>
            </button>
          </div>
          <div className="hidden lg:block text-right whitespace-nowrap group relative cursor-pointer font-bold" onClick={handleResetApp}>
            <div className="group-hover:scale-95 transition-transform font-bold">
              <p className="text-[10px] font-bold text-blue-300 tracking-wide mb-0.5 font-bold">Premium lab</p>
              <p className="text-xs font-bold tracking-tighter underline decoration-blue-500 underline-offset-2 font-bold">Made by Taeo</p>
            </div>
            
            <div className="absolute right-0 top-full mt-3 hidden group-hover:block z-[100] animate-in fade-in slide-in-from-top-2 duration-200 cursor-default font-bold">
              <div className="bg-slate-900 border border-slate-800 rounded-2xl shadow-2xl p-4 min-w-[240px] text-center relative font-bold">
                <p className="text-yellow-400 font-black text-[13px] tracking-tight mb-1.5 font-bold uppercase">카카오뱅크 : 3333-06-1042475</p>
                <p className="text-slate-500 text-[10px] font-bold tracking-tighter font-bold">후원 감사합니다.</p>
                <div className="absolute -top-1.5 right-6 w-3 h-3 bg-slate-900 rotate-45 border-t border-l border-slate-800 font-bold font-bold"></div>
              </div>
            </div>
          </div>
        </div>
        {isMobileMenuOpen && (
          <div className="fixed inset-0 z-[100] bg-white flex flex-col p-6 animate-in slide-in-from-right duration-300 overflow-y-auto font-bold font-bold">
            <div className="flex justify-between items-center mb-12 font-bold font-bold font-bold">
              <h1 className="text-[24px] font-bold tracking-tighter uppercase leading-none font-bold font-bold" onClick={handleResetApp}>
                 <span className="text-slate-900 font-bold">OPIc </span>
                 <span className="text-blue-600 font-bold">IH</span>
                 <span className="text-slate-900 font-bold"> MASTER</span>
              </h1>
              <button onClick={() => setIsMobileMenuOpen(false)} className="p-2 text-slate-600 font-bold font-bold font-bold">
                <i className="fas fa-times text-3xl font-bold font-bold font-bold"></i>
              </button>
            </div>
            <div className="flex flex-col gap-4 font-bold font-bold">
              {[
                {id: 'analysis', label: '유형 분석'}, 
                {id: 'survey', label: '서베이 설정'}, 
                {id: 'masterAnswers', label: '만능 스크립트'}, 
                {id: 'fillers', label: '필러 표현'}, 
                {id: 'mocktest', label: '실전 모의고사'},
                {id: 'opicAI', label: '오픽AI'}
              ].map(tab => (
                <button key={tab.id} onClick={() => { setActiveTab(tab.id); setIsMobileMenuOpen(false); }} className={`w-full text-left p-5 rounded-2xl font-bold text-xl transition-all flex justify-between items-center font-bold ${activeTab === tab.id ? 'bg-blue-600 text-white font-bold' : 'bg-slate-50 text-slate-600'}`}>{tab.label}{activeTab === tab.id && <i className="fas fa-check text-sm font-bold font-bold font-bold"></i>}</button>
              ))}
            </div>
          </div>
        )}
      </header>
      <main className="flex-1 w-full pb-40 font-bold">
        {activeTab === 'analysis' && renderAnalysis()}
        {activeTab === 'survey' && renderSurvey()}
        {activeTab === 'masterAnswers' && renderMasterAnswers()}
        {activeTab === 'fillers' && renderFillers()}
        {activeTab === 'mocktest' && renderMockTest()}
        {activeTab === 'opicAI' && renderOpicAI()}
      </main>
      <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" />
      <style>{`
        @import url('https://cdn.jsdelivr.net/gh/orioncactus/pretendard/dist/web/static/pretendard.css');
        :root { --font-pretendard: 'Pretendard', -apple-system, BlinkMacSystemFont, system-ui, Roboto, sans-serif; }
        * { font-family: var(--font-pretendard) !important; font-style: normal !important; }
        body { background-color: #FDFDFD; }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        .animate-in { animation: fadeIn 0.6s ease-out forwards; }
      `}</style>
    </div>
  );
};

export default App;

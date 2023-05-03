### 😎 DooDuZ
Junior BackEnd developer

🛠️ Stack
---
<img src="https://img.shields.io/badge/JAVA-007396?style=for-the-badge&logo=java&logoColor=white"> <img src="https://img.shields.io/badge/Spring%20Boot-6DB33F?style=for-the-badge&logo=Spring%20Boot&logoColor=white">  
<img src="https://img.shields.io/badge/javascript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black"> <img src="https://img.shields.io/badge/react-61DAFB?style=for-the-badge&logo=react&logoColor=black"> <img src="https://img.shields.io/badge/html-E34F26?style=for-the-badge&logo=html5&logoColor=white"> <img src="https://img.shields.io/badge/css-1572B6?style=for-the-badge&logo=css3&logoColor=white">  
<img src="https://img.shields.io/badge/mysql-4479A1?style=for-the-badge&logo=mysql&logoColor=white"> <img src="https://img.shields.io/badge/aws-orange?style=for-the-badge&logo=amazon%20aws&logoColor=white"> <img src="https://img.shields.io/badge/github-181717?style=for-the-badge&logo=github&logoColor=white">

### BOJ profile
[![Solved.ac Profile](http://mazassumnida.wtf/api/v2/generate_badge?boj=sin9158)](https://solved.ac/sin9158/)

### 주요 소스

##### Project MoKkoJi
###### 좋아요/싫어요 입력 및 리턴

```java
@Transactional
public boolean setlikes(LikesDto likesDto){
    String memail = memberService.getloginMemail();
    int mno = memberService.getmno(memail);
    int bno = likesDto.getBno();
    boolean trueOrFalse = likesDto.isLikeInfo();
    Optional<LikesEntity> optional = likesRepository.findByRecords(bno, mno);

    //게시물번호 회원로그인 확인
    if(bno <= 0 && memail==null){return false;}

    //레코드를 찾을 수 없을 때 , 게시물 번호가 정상적이며 로그인이 되어 있을 때
    if(!optional.isPresent()){
        //엔티티 저장
        try{
            LikesEntity likesEntity= likesRepository.save(likesDto.toEntity());
            Optional<MemberEntity>Moptional= memberRepository.findById( mno );
            Optional<BoardEntity> Boptional= boardRepository.findById( bno );
            // null 확인
            if(Boptional.isPresent() && Moptional.isPresent()){
                MemberEntity memberEntity = Moptional.get();
                BoardEntity boardEntity   = Boptional.get();
                //교차저장
                memberEntity.getLikesEntityList().add(likesEntity);
                boardEntity.getLikesEntityList().add(likesEntity);
                likesEntity.setMemberEntity(memberEntity);
                likesEntity.setBoardEntity(boardEntity);
                return true;
            }
        }catch(Exception e){
            System.out.println("좋아요 set 오류" + e);
        }
    }
    //있던 레코드
    else{
        LikesEntity entity= optional.get();
        try{
            if(entity.isLikeInfo() == trueOrFalse){likesRepository.delete(entity);}
            else{entity.setLikeInfo(!entity.isLikeInfo());}
            return true;
        }catch (Exception e){
            System.out.println("좋아요 상태 변경 오류" + e);
        }
    }
    return false;
}

/*
    20221215 지웅/유정
    게시물 열람 시 좋아요 숫자 계산 후 return
    getBoard에서 호출
    @Param bno
         게시물 번호
    @return int[2] arr
         arr[0] = 좋아요 수 / arr[1] = 싫어요 수
*/
public int[] likesCount( int bno ){
    Optional<BoardEntity> optional = boardRepository.findById(bno);
    if( !optional.isPresent() ){ return null; }
    BoardEntity boardEntity = optional.get();

    List<LikesEntity> list = boardEntity.getLikesEntityList();

    int countLikes = 0;
    int countDislikes = 0;

    for(LikesEntity entity : list){
        if(entity.isLikeInfo()){
            countLikes++;
        }else{
            countDislikes++;
        }
    }
    int[] arr = new int[2];
    arr[0] = countLikes;
    arr[1] = countDislikes;

    return arr;
}


```

##### Project MAMIN  
###### WebSocket Endpoint
```java
// 지웅 20221031 23:50 추가
  // 유저 수 4명 이상일 시 room->index.jsp로 내보내기
public void UserOverflow(Session session) throws IOException {
  JSONObject obj = new JSONObject();
  obj.put("function_name", "exit");
  session.getBasicRemote().sendText(obj.toString());
}

public void UserOverflow(Session session, String m_id) throws IOException {
  JSONObject obj = new JSONObject();
  obj.put("function_name", "duplicated");
  session.getBasicRemote().sendText(obj.toString());
}

// 지웅 20221030 유저 입장
@OnOpen
public void OnOpen( Session session, @PathParam("m_id") String m_id ) throws IOException  {
  MemberDto dto = MemberDao.getInstance().getMember(m_id);//11/03 장군 클라이언트 소켓 들어왔을때 채팅창 알림 전송용
  JSONObject object = new JSONObject();
  object.put("m_nick", dto.getM_nick());
  object.put("type","open" );
  if(clients.containsValue(m_id)) {
    UserOverflow(session, m_id);
  }		
  if(clients.size()>=4) {
    UserOverflow(session);
  }
  clients.put(session, m_id);
  for(Session s: clients.keySet()) {
    s.getBasicRemote().sendText(object.toString());
  }
  getPlayerInfo(m_id);
}
// 지웅 20221030 유저 퇴장
@OnClose
public void OnClose( Session session ) throws IOException {
  MemberDto dto = MemberDao.getInstance().getMember(clients.get(session));//11/03 장군 클라이언트 소켓 나갔을때 채팅창 알림 전송용
  JSONObject object = new JSONObject();
  object.put("m_nick", dto.getM_nick());
  object.put("type","close" );

  for(int i = 0; i<players.size(); i++) {
    if(players.get(i).getM_id().equals(clients.get(session))) {
      players.remove(i);
      break;
    }
  }
  clients.remove(session);
  for(Session s: clients.keySet()) {
    s.getBasicRemote().sendText(object.toString());
  }
  getPlayerInfo();
}

// 지웅 20221030 js에서 send()함수로 서버 접근 시 서버 접속 중인 인원들에게 줄 정보를 js의 OnMessage로 전송
@OnMessage
public void OnMessage( Session session, String object ) throws IOException{
  if(object.equals("\"getPlayersInfo\"")) {
    getPlayerInfo();
    return;
  }else if(object.contains("\"roomdata\":\"ready")) {	// parser 배우기 전 작성 -> 스트링에 특정 데이터 포함된 경우
    setready(object);
  }		
  synchronized (session) {
    for(Session s : clients.keySet()) {
      s.getBasicRemote().sendText(object);	// 문자열 받아서 모두에게 전송
    }			
  } 
}

```

### Contact
✉️ sin9158@naver.com

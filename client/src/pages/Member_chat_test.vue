<template>

  채팅 페이지
  <p>UserName: <input v-model="userName" type="text"></p>
  <section style="background-color: #eee;">
    <div class="container py-5">

      <div class="row d-flex justify-content-center">
        <div class="col-md-8 col-lg-6 col-xl-4">

          <div class="card" id="chat1" style="border-radius: 15px;">
            <div
              class="card-header d-flex justify-content-between align-items-center p-3 bg-info text-white border-bottom-0"
              style="border-top-left-radius: 15px; border-top-right-radius: 15px;">
              <i class="fas fa-angle-left"></i>
              <p class="mb-0 fw-bold">Live chat</p>
              <i class="fas fa-times"></i>
            </div>
            <div class="card-body">

              <div v-for="(item, idx) in recvList" :key="idx" class="d-flex flex-row justify-content-end mb-4">
                <div class="p-3 me-3 border" style="border-radius: 15px; background-color: #fbfbfb;">
                  <p>NAME : {{ item.userName }} </p>
                  <p class="small mb-0">{{ item.content }}</p>
                </div>
                <img src="https://mdbcdn.b-cdn.net/img/Photos/new-templates/bootstrap-chat/ava2-bg.webp" alt="avatar 1"
                  style="width: 45px; height: 100%;">
              </div>

              <div class="form-outline">
                <textarea  v-model="message" type="text" @keyup="sendMessage" class="form-control" id="textAreaExample" rows="4">gd
                </textarea>

                <label class="form-label" for="textAreaExample">

                </label>
              </div>
            </div>
          </div>
        </div>
      </div>

    </div>
  </section>
</template>

<script>
import Stomp from 'webstomp-client'
import SockJS from 'sockjs-client'

export default {

  data() {
    return {
      userName: "",
      message: "",
      recvList: []
    }
  },
  created() {
    this.connect()
  },
  methods: {
    sendMessage (e) {
      if(e.keyCode === 13 && this.userName !== '' && this.message !== ''){
        this.send()
        this.message = ''
      }
    },    
    send() {
      console.log("Send message:" + this.message);
      if (this.stompClient && this.stompClient.connected) {
        const msg = { 
          userName: this.userName,
          content: this.message 
        };
        this.stompClient.send("/receive", JSON.stringify(msg), {});
      }
    },    
    connect() {
      const serverURL = "http://localhost:8080"
      let socket = new SockJS(serverURL);
      this.stompClient = Stomp.over(socket);
      console.log(`소켓 연결을 시도합니다. 서버 주소: ${serverURL}`)
      this.stompClient.connect(
        {},
        frame => {
          // 소켓 연결 성공
          this.connected = true;
          console.log('소켓 연결 성공', frame);

          this.stompClient.subscribe("/send", res => {
            console.log('구독으로 받은 메시지 입니다.', res.body);

            // 받은 데이터를 json으로 파싱하고 리스트에 넣어줍니다.
            this.recvList.push(JSON.parse(res.body))
          });
        },
        error => {
          // 소켓 연결 실패
          console.log('소켓 연결 실패', error);
          this.connected = false;
        }
      );
    }
  }
}
</script>
<template>
  <div id="app"  class="grid">
    <section class="item1">
      <div class="item1-body">
        <div class="navbar">
            <div class="logo-box">
              <img src="./assets/logo.png" alt="Logo" class="logo-img">
            </div>
          <a><router-link to="/"> QUIZ</router-link></a>
          <a v-if="!isLogged"><router-link to="/register">REGISTER</router-link></a>
          <a><router-link to="/aboutUs">ABOUT US</router-link></a>
          <div class="dropdown" v-if="isLogged">
            <button class="dropbtn">
              MY PAGE
              <img src="./assets/icon/down.png" alt="down">
            </button>
            <div class="dropdown-content">
              <div class="header"></div>
              <a><router-link to="/MySpace" >My Profile</router-link></a>
              <a  v-if="user_Role ==='Student'"><router-link to="/myScores">My Scores</router-link></a>
              <a  v-if="user_Role ==='Student'"><router-link to="/myRates">My Rates</router-link></a>
              <a v-if="user_Role ==='Admin'"><router-link to="/AllUsers">Manage Users</router-link></a>
              <a  v-if="user_Role ==='Admin'"><router-link to="/AllQuiz">Manage Quizzes</router-link></a>
              <a  v-if="user_Role !=='Student'"><router-link to="/AllScores">All Scores</router-link></a>
              <a  v-if="user_Role !=='Student'"><router-link to="/AllRates">All Rates</router-link></a>
              <a  v-if="user_Role !=='Student'"><router-link to="/CreateNewQuiz">Create Quiz</router-link></a>
            </div>
          </div>
        </div>
      </div>
    </section>
    <section class="item2">
      <div class="item2-body">
        <div class="login-cell">
          <table class="center">
            <tr>
              <td><span>{{message}}</span><span class="error">{{errorMessage}}</span> &nbsp;<input class="btn-logout" type="button" @click="submitLogout" value="logout" v-if="isLogged"></td>
            </tr>
            <tr>
              <td v-if="!isLogged"><input value="email" type="email" name="user-log" v-model="email" placeholder="Email" >&nbsp;<input value="password" type="password" name="user-log" v-model="password" placeholder="Password">&nbsp;<input class="btn-login" type="button" @click="submitForm()" value="login"></td>
            </tr>
            <tr>
              <td v-if="!isLogged"><span>Har inget konto? </span><a><router-link to="/register">Registrera!</router-link></a></td>
            </tr>
          </table>
        </div>
      </div>
    </section>
    <section class="item3">
      <div class="item3-body">
        <div class="body-routers">
          <router-view />
        </div>
      </div>

    </section>
    <section class="item4">
      <div class="item4-body">
        <p>Copyright (c) 2020 by Aisha, Anton, Halim, Linus.</p>
      </div>
    </section>

   </div>
</template>
<script>
  import AuthServices from './services/ApiServices';
  export default {
    name: "App",
    data: function () {
      return {
        email:'',
        password:'',
        message:'',
        errorMessage:'',
        isLogged: false,
        userID: 0,
        user_Role:'',
      }
    },
      methods: {
        mobilMenu () {
          const x = document.getElementById("links");
          if (x.style.display === "block") {
            x.style.display = "none";
          } else {
            x.style.display = "block";
          }
        },
        async submitForm() {
          const response = await AuthServices.login({
            email: this.email,
            password: this.password
          });
          if (response.data.isSessionCreated){
            this.errorMessage ='';
            const jsonObject = '{"message": "'+response.data.message+'", "fullName": "'+response.data.fullName+'", "userRole": "'+response.data.userRole+'", "userId": "'+response.data.userId+'"}';
            let obj = JSON.parse(jsonObject);
            let jsonUser_serialized = JSON.stringify(obj);
            sessionStorage.setItem("userLogged", jsonUser_serialized);
            this.message= JSON.parse(sessionStorage.getItem('userLogged')).message+", "+JSON.parse(sessionStorage.getItem('userLogged')).fullName+"!";
            this.isLogged = true;
            this.userID= parseInt(JSON.parse(sessionStorage.getItem('userLogged')).userId);
            this.user_Role= JSON.parse(sessionStorage.getItem('userLogged')).userRole;
            this.email = '';
            this.password = '';
            location.reload();
          }
          else {
            this.message='';
            this.errorMessage = response.data.message;
            this.email = '';
            this.password = '';
          }
        },
        async submitLogout () {
          const response = await AuthServices.logout({
           sessionStatus: this.isLogged,
          });
          sessionStorage.clear();
          this.isLogged = false;
          this.userID= 0;
          this.user_Role= '';
          this.errorMessage ='';
          this.message = response.data.message;
          await this.$router.push({ path: '/' });
          location.reload();
        },

      },
    mounted(){
        if (JSON.parse(sessionStorage.getItem('userLogged')).userId && parseInt(JSON.parse(sessionStorage.getItem('userLogged')).userId)>=1){
          this.isLogged = true;
          this.message= JSON.parse(sessionStorage.getItem('userLogged')).message+", "+JSON.parse(sessionStorage.getItem('userLogged')).fullName+"!";
          this.userID= parseInt(JSON.parse(sessionStorage.getItem('userLogged')).userId);
          this.user_Role= JSON.parse(sessionStorage.getItem('userLogged')).userRole;
        }
    }
  }
</script>

<style>
/* style*/
  [class^="item"] {
    text-align: center;
  }
  .grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    grid-template-rows: repeat(4, auto);
    grid-template-areas:
            "header header header header"
            "menu menu menu menu"
            "sidebar sidebar sidebar sidebar"
            "footer footer footer footer";
    grid-auto-flow: dense;
    grid-gap: 5px;
  }
  .item1 {
    grid-area: header;
  }

  .item2 {
    grid-area: menu;
  }

  .item3 {
    grid-area: sidebar;
    display:flex;
  }

  .item4 {
    grid-area: footer;
  }
  body{
    font-family: Calibri, monospace;
  }
  .error{
    color: red;
  }
  .login a {
    color: #02b3b3;
    text-decoration: none;
  }
.login a:hover {
    color: #0b5b5b;
}
.item3-body{
  border: 3px solid #cccc;
  width: 100%;
  display: table;
  margin: auto;
  background: url('./assets/qui.jpg') no-repeat;
  background-size: 100% 100%;
}
span{
  font-family: Calibri, monospace;
  font-weight: normal;
  color: wheat;
}
  .item2-body{
    border: 3px solid #cccc;
    width: 100%;
    display: table;
    margin: auto;
  }
  .login-cell{
    height: 50px;
    width: 100%;
    display: table-cell;
    text-align: center;
    vertical-align: middle;
    background: #0f122d;
  }
  .body-routers{
    display: table-row;
    text-align: center;
  }
a {
  color: #0b5b5b;
  text-decoration: none;
}
a:hover {
  color: #02b3b3;
}
.center {
  width: 100%;
  font: inherit;
  border-collapse: collapse;
  color: rgba(6, 25, 45, 0.6);
}

.center th {
  text-transform: uppercase;
  text-align: center;
  background: #44475C;
  color: #FFF;
  padding-top: 5px;
}

.center td {
  padding-top: 5px;
  text-align: center;
}


  .item1-body{
    width: 100%;
    border: 3px solid #ccc;
    display: inline-block;
  }
  .logo-box{
    float: left;
    overflow: hidden;
    height: 100%;
  }
  .logo-img{
    padding-left: 3px;
    padding-top: 2px;
    max-width: 100px;
    max-height: 100px;
    display: inline-block;
  }
  .item4-body{
    border: 3px solid #ccc;
    display: inline-block;
    width: 100%;
  }
  button{
    margin-top: 5px;
    border: none;
    background: transparent;
    width: 40px;
    height: 40px;
    cursor: pointer;
    text-align: center;
  }

/* Menu Mobile & Desktop*/
* {
  box-sizing: border-box;
}
.navbar {
  overflow: hidden;
  background-color: #0f122d;
  font-family: Calibri, monospace;
  font-size: large;
}

.navbar a {
  float: right;
  font-size: 14px;
  font-weight: bold;
  color: white;
  text-align: center;
  padding: 7px 0;
  text-decoration: none;
  width: 85px;
}

.dropdown {
  float: right;
  overflow: hidden;
}

.dropdown .dropbtn {
  font-weight: bold;
  font-size: 14px;
  border: none;
  outline: none;
  color: white;
  padding: 8px 0;
  background-color: inherit;
  font-family: inherit;
  margin-bottom: 2px;
  width: 85px;
}

.navbar a:hover, .dropdown:hover, .dropbtn:hover {
  background-color: #ddd;
  color: black;
}

.dropdown-content {
  display: none;
  position: absolute;
  background-color: #f9f9f9;
  min-width: 85px;
  box-shadow: 0px 8px 16px 0px rgba(0,0,0,0.2);
  z-index: 1;
}

.dropdown-content a {
  font-weight: normal;
  float: none;
  color: black;
  padding: 4px 0;
  text-decoration: none;
  display: block;
  text-align: left;
}

.dropdown-content a:hover {
  background-color: #ddd;
}
.header {
  background: #0f122d;
  padding-top: 5px;
  padding-bottom: 7px;
  color: white;
}

.dropdown:hover .dropdown-content {
  display: block;
}
/* finish here */
.btn-login, .btn-logout {
  background-color: #222222;
  font-family: "Times New Roman", monospace;
  font-weight: bold;
  color: #02b3b3;
  border: 1px solid rgb(7, 172, 172);
  border-radius: 4px;
  width: 50px;
  height: 30px;
  cursor: pointer;
}
.btn-login:hover {
  border: 1px solid black;
  background-color: #0707bf;
  color: white;
}
.btn-logout:hover {
  border: 1px solid black;
  background-color: #d40404;
  color: white;
}
input[type=email], input[type=password] {
  padding: 10px;
  margin-top: 2px;
  margin-bottom: 2px;
  border: 1px solid rgb(7, 172, 172);
  border-radius: 4px;
  box-sizing: border-box;
  resize: vertical;
  background: rgba(5, 5, 5, 0.9);
  color: wheat;
  font-family: Calibri, monospace;
  font-weight: bold;
  height: 30px;
  cursor: pointer;
  width: 100%;
}
.md-toolbar + .md-toolbar {
  margin-top: 16px;
}
h5{
  margin: 0;
}
  /* Mobile */
  @media screen and (max-width: 400px) {
  }
  /* Tablet */
  @media screen and (min-width: 768px) and (max-width: 1024px) {
    .item3-body{
      background: url('./assets/zoom.webp') no-repeat;
      background-size: 100% 100%;
    }
    .btn-login, .btn-logout {
      width: 50px;
      height: 30px;
    }
    input[type=email], input[type=password] {
      width: 45%;
    }
    .logo-box{
      float: left;
      overflow: hidden;
    }
    .logo-img{
      padding-left: 3px;
      max-width: 130px;
      max-height: 130px;
      display: inline-block;
    }
    .dropdown .dropbtn {
      padding: 14px 0;
      width: 110px;
      margin-bottom: 14px;
      font-size: 16px;
    }
    .navbar a {
      padding: 10px 0;
      width: 110px;
      font-size: 16px;
    }
    .dropdown-content a {
      text-align: left;
      display: block;
      padding: 6px 0;
    }
    .dropdown-content {
      min-width: 110px;
      box-shadow: 0px 8px 16px 0px rgba(0,0,0,0.2);
      z-index: 1;
    }

  }
  /* Desktop */
  @media screen and (min-width: 1025px) {
    .item3-body{
      background: url('./assets/qui.jpg') no-repeat;
      background-size: 100% 100%;
      width: 70%;
      height: 500px;
    }
    .item2-body{
      border: 3px solid #cccc;
      width: 70%;
    }
    .item1-body{
      display: inline-block;
      text-align: center;
      width: 70%;
      margin-top: 5px;
    }
    .logo-box{
      float: left;
      overflow: hidden;
      max-width: 128px;
      max-height: 100px;
    }
    .logo-img{
      padding: 4px;
      max-width: 140px;
      max-height: 140px;
      display: block;
    }
    .item4-body{
      width: 70%;
      background-color: #0f122d;
        color: wheat;
        font-family: Calibri, monospace;
        font-weight: bold;
        font-size: small;
    }
    .btn-login, .btn-logout  {
      width: 50px;
      height: 30px;
    }
    input[type=email], input[type=password] {
      width: 200px;
    }
    .dropdown .dropbtn {
      padding: 18px 0;
      width: 120px;
      margin-bottom: 18px;
      font-size: 16px;
    }
    .navbar a {
      padding: 12px 0;
      width: 120px;
      font-size: 16px;
    }
    .dropdown-content a {
      display: block;
      padding: 10px 0;
      //font-weight: bold;
    }
    .dropdown-content {
      min-width: 120px;
      box-shadow: 0px 8px 16px 0px rgba(0,0,0,0.2);
      z-index: 1;
    }

  }


</style>

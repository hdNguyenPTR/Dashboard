<template>
  <div class="sign-up">
    <img alt="logo" src="../assets/logo_white.jpg">
    <p>Let's create a new account !</p>
    <form class="form-signin">
      <input type="text" class="form-control" v-model="email" placeholder="Email">
      <input type="password" class="form-control" v-model="password" placeholder="Password">
      <input type="password" class="form-control" v-model="password2" placeholder="Repeat Password">
      <br>
      <button type="button" class="btn btn-success btn-lg btn-block" @click="signUp">Sign Up</button>
    </form>
    <span>
      or go back to
      <router-link to="/login">login</router-link>
    </span>
  </div>
</template>

<script>
import firebase from "firebase";

export default {
  name: "signUp",
  data() {
    return {
      email: "",
      password: "",
      password2: ""
    };
  },
  methods: {
    signUp: function() {
      if (this.password === this.password2) {
        firebase
          .auth()
          .createUserWithEmailAndPassword(this.email, this.password)
          .then(
            user => {
              alert("You have created successfully!");
              this.$router.replace("home");
            },
            err => {
              alert("Oops. " + err.message);
            }
          );
      } else {
        alert("Password is not matching.");
      }
    }
  }
};
</script>

<style scoped>
html,
body {
  height: 100%;
}

body {
  display: -ms-flexbox;
  display: flex;
  -ms-flex-align: center;
  align-items: center;
  padding-top: 40px;
  padding-bottom: 40px;
  background-color: #f5f5f5;
}

.form-signin {
  width: 100%;
  max-width: 330px;
  padding: 15px;
  margin: auto;
}
.form-signin .checkbox {
  font-weight: 400;
}
.form-signin .form-control {
  position: relative;
  box-sizing: border-box;
  height: auto;
  padding: 10px;
  font-size: 16px;
}
.form-signin .form-control:focus {
  z-index: 2;
}
.form-signin input[type="email"] {
  margin-bottom: -1px;
  border-bottom-right-radius: 0;
  border-bottom-left-radius: 0;
}
.form-signin input[type="password"] {
  margin-bottom: -1px;
  border-top-left-radius: 0;
  border-top-right-radius: 0;
}
</style>
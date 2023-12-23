<template>
  <div>
    <h1>Login Page</h1>
    <form @submit.prevent="login">
      <label for="username">Username</label>
      <input type="text" id="username" v-model="username" required>
      <br>
      <label for="password">Password</label>
      <input type="password" id="password" v-model="password" required>
      <br>
      <button type="submit">Login</button>
    </form>
    <p v-if="errorMessage" style="color: red;">{{ errorMessage }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      username: '',
      password: '',
      errorMessage: '',
    };
  },
  methods: {
    async login() {
      try {
        const response = await fetch('http://localhost:9876/user/login', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            username: this.username,
            password: this.password,
          }),
        });

        const data = await response.json();

        if (data.code === -1) {
          // 登录失败
          this.errorMessage = data.message;
        } else {
          // 登录成功
          const uuid = data.data;
          localStorage.setItem('uuid', uuid);
          this.$router.push('/home');
        }
      } catch (error) {
        console.error('Login error:', error);
      }
    },
  },
};
</script>


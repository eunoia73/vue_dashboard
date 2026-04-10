<template>
  <div id="app">
    <!-- 네비게이션 메뉴 -->
    <nav class="navbar">
      <div class="nav-container">
        <div class="logo">📊 실시간 정보 대시보드</div>
      </div>
    </nav>

    <!-- 라우트 매칭된 컴포넌트가 렌더링되는 영역 -->
    <main class="main-content">
      <router-view />
    </main>

    <!-- 공통 푸터 -->
    <footer class="footer">
      <p>&copy; 2026 My App</p>
    </footer>
  </div>
</template>

<script setup></script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: #f5f7fa;
}

#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* 네비게이션 */
.navbar {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  position: sticky;
  top: 0;
  z-index: 100;
}

.nav-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 70px;
}

.logo {
  font-size: 24px;
  font-weight: bold;
  color: white;
  letter-spacing: 1px;
}

/* 메인 컨텐츠 */
.main-content {
  flex: 1;
  max-width: 1200px;
  width: 100%;
  margin: 0 auto;
  padding: 40px 20px;
}

/* 푸터 */
.footer {
  background: #2d3748;
  color: white;
  text-align: center;
  padding: 30px 20px;
  margin-top: auto;
}

.footer p {
  opacity: 0.8;
  font-size: 14px;
}
</style>

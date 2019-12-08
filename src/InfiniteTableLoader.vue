<template>
  <div class="loader-container">
    <div class="loader-label">
      <div>{{message}}</div>
    </div>
    <div class="loader" />
  </div>
</template>

<script>
export default {
  name: "InfiniteTableLoader",
  props: ["message"]
};
</script>

<style scoped>
.loader-container {
  color: white;
  margin: auto;
  position: relative;
  display: flex;
}
.loader-label {
  position: absolute;
  left: 1em;
  right: 1em;
  top: 0;
  bottom: 0;
  display: flex;
}
.loader-label > div {
  margin: auto;
  text-align: center;
}
.loader,
.loader:after {
  border-radius: 50%;
  width: 10em;
  height: 10em;
}
.loader {
  margin: auto;
  font-size: 10px;
  position: relative;
  text-indent: -9999em;
  border-top: 1.1em solid rgba(255, 255, 255, 0.2);
  border-right: 1.1em solid rgba(255, 255, 255, 0.2);
  border-bottom: 1.1em solid rgba(255, 255, 255, 0.2);
  border-left: 1.1em solid #ffffff;
  -webkit-transform: translateZ(0);
  -ms-transform: translateZ(0);
  transform: translateZ(0);
  -webkit-animation: load8 1.1s infinite linear;
  animation: load8 1.1s infinite linear;
}
@-webkit-keyframes load8 {
  0% {
    -webkit-transform: rotate(0deg);
    transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}
@keyframes load8 {
  0% {
    -webkit-transform: rotate(0deg);
    transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}
</style>

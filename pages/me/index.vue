<template>
	<div class="me">
		<header class="header" >
			<img src="../../static/image/nature.jpg" alt="" class="header_bg">
			<img :src="data.userinfo.avatar" alt="avatar" class="avatar">
			<div class="content">
				<!-- @click='login' -->
				<div class="nologin" v-if="!data.userinfo.name" >您好</div>
				<div class="name"  v-else>{{data.userinfo.name}}</div>
			</div>
		</header>
	</div>
</template>

<script setup>
import { reactive } from "vue";
	const data = reactive({
		userinfo: {
			name: '',
			avatar: 'https://cube.elemecdn.com/3/7c/3ea6beec64369c2642b92c6726f1epng.png'
		}
	})
	const login = () => {
      // 展示加载框
      uni.showLoading({
        title: '加载中',
      });
      uni.getUserProfile({
        desc: '登录后可同步数据',
        success: async (obj) => {
          console.log('obj', obj);
          // 调用 action ，请求登录接口
          // await this.login(obj);
          uni.login({
            provider: 'weixin',
            success: (res) => {
              console.log('res-login', res);
              this.code = res.code;
              console.log('code', res.code);
              if (res.errMsg == 'login:ok') {
                uni
                  .request({
                    url:
                      'https://justinli.love/wxh5/wx/user/' +
                      'wx55822xxxx75e422' +
                      '/login/',
                    data: {
                      code: this.code,
                    },
                  })
                  .then((res) => {
                  //获取到 openid 和 session_k后，自己的逻辑
                    console.log('授权登录', res[1].data);
                    console.log(res[1].data.openid);
                    console.log(res[1].data.session_key);
                    // DoSomeThing.................
                  });
                console.log('res', res);
              }
            },
          });
        },
        fail: () => {
          uni.showToast({
            title: '授权已取消',
            icon: 'error',
            mask: true,
          });
        },
        complete: () => {
          // 隐藏loading
          uni.hideLoading();
        },
      });
	}
</script>

<style lang="scss" scoped>
	.me {
		padding: 30px 10px;
			.header {
				height: 80px;
				border: 1px solid #ccc;
				border-radius: 5px;
				display: flex;
				align-items: center;
				justify-content: space-between;
				// background-image: url('~@/static/image/nature.jpg');
				background-position: 100%;
				background-repeat: no-repeat;
				background-size: cover;
				font-size: 18px;
				color: #fff;
				position: relative;
				.header_bg{
					position: absolute;
					width: 100%;
					height: 80px;
					object-fit: fill;
				}
				.avatar {
					height: 50px;
					width: 50px;
					margin: 0 20px;
					border-radius: 50%;
					z-index: 1;
				}
				.content {
					flex: 1;
					z-index: 1;
				}
			}
	}
</style>
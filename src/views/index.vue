<style scoped>
    html,body{
        height: 100%;
        width: 100%;
        padding: 0;
        margin: 0;
    }

    .login-form {
        margin: 0 auto;
        position: absolute;
        top: 200px;
        width: 100%;
    }

    .login-card {
        background-color: rgba(120,120,120,0.1)!important;background-color: #000;filter:Alpha(opacity=10);
    }

    .login-bottom {
        position: absolute;
        bottom: 0px;
        left: 0px;
        right: 0px;
        height: 80px;
        text-align: right;
    }

    .login-bottom img{
        height: 44px;
        margin-right: 32px
    }
    
    .back {
        background: url("./images/back.png") no-repeat fixed top;;
        background-size: cover;
        min-height: 800px;
        min-width: 1024px;
        width: 100%;
        height: 100%;
        position: relative;
    }
</style>
<template>
    <div class="back">
        <canvas></canvas>
        <div class="login-form">
            <div style="width: 400px;margin: 0 auto;position: relative">
                <Card style="" dis-hover class="login-card">
                    <h1 style="text-align: center;margin-bottom: 16px">信息共享机制与平台</h1>
                    <Form ref="formCustom" :model="formCustom" :rules="ruleCustom" label-position="left">
                        <FormItem prop="user">
                            <i-input type="text" v-model="formCustom.username" placeholder="用户名">
                            <Icon type="ios-person-outline" slot="prepend"></Icon>
                            </i-input>
                        </FormItem>
                        <FormItem prop="password">
                            <i-input type="password" v-model="formCustom.password" placeholder="密码">
                            <Icon type="ios-locked-outline" slot="prepend"></Icon>
                            </i-input>
                        </FormItem>
                        <FormItem style="text-align: center">
                            <Button @click="login('formCustom')">登录</Button>
                        </FormItem>
                    </Form>
                </Card>
                <div style="position: absolute;bottom: 5px;right: 8px;color: #777777">版本:V0.0.01</div>
            </div>
        </div>
        <div class="login-bottom">
        </div>
    </div>
</template>
<script type="text/ecmascript-6">

    import {mapMutations} from 'vuex';
    export default {
        data() {
            return {
                formCustom : {
                    username : '',
                    password : ''
                },
                ruleCustom: {
                    username: [
                        { required: true,message:'请输入用户名', trigger: 'blur' }
                    ],
                    password: [
                        { required: true,message:'请输入密码', trigger: 'blur' }
                    ]
                }
            };
        },
        mounted : function () {
            try {
                this.renderBackground();
            } catch (e) {
            }
        },
        methods : {
            ...mapMutations({
                changeLoginInfo:'CHANGELOGININFO'
            }),
            login(name){
                var me=this;
                me.$refs[name].validate((valid) => {
                    if (valid) {
                        let formData = new FormData();
                        formData.append('username',me.formCustom.username);
                        formData.append('password',me.formCustom.password);
                        me.$http.post('/login',formData).then((response) => {
                            const info={};
                            console.log(response)
                            info.authToken = response.headers.auth_token;
                            info.username = response.data.data &&response.data.data.name;
                            info.loginId = response.data.data &&response.data.data.loginId;
                            window.localStorage.loginInfo=JSON.stringify(info);
                            this.changeLoginInfo(info);
                            me.$router.push({path:'/main'});
                        }).catch((res) => {
                            debugger
                            me.$Message.info('登录失败');
                        })
                    } else {
                        me.$Message.error('表单验证失败!');
                    }
                })
            },
            renderBackground: function () {
                var canvas = document.querySelector('canvas'),
                    ctx = canvas.getContext('2d')
                canvas.width = window.innerWidth-5;
                canvas.height = window.innerHeight-5;
                window.onresize = function(){
                    canvas.width = window.innerWidth < 1024 ? 1024 : (window.innerWidth-5);
                    canvas.height = window.innerHeight < 800 ? 800 : (window.innerHeight-5);
                }
                ctx.lineWidth = .3;
                ctx.strokeStyle = (new Color(150)).style;

                var mousePosition = {
                    x: 30 * canvas.width / 100,
                    y: 30 * canvas.height / 100
                };

                var dots = {
                    nb: 250,
                    distance: 100,
                    d_radius: 150,
                    array: []
                };

                function colorValue(min) {
                    return Math.floor(Math.random() * 255 + min);
                }

                function createColorStyle(r,g,b) {
                    return 'rgba(' + r + ',' + g + ',' + b + ', 0.8)';
                }

                function mixComponents(comp1, weight1, comp2, weight2) {
                    return (comp1 * weight1 + comp2 * weight2) / (weight1 + weight2);
                }

                function averageColorStyles(dot1, dot2) {
                    var color1 = dot1.color,
                        color2 = dot2.color;

                    var r = mixComponents(color1.r, dot1.radius, color2.r, dot2.radius),
                        g = mixComponents(color1.g, dot1.radius, color2.g, dot2.radius),
                        b = mixComponents(color1.b, dot1.radius, color2.b, dot2.radius);
                    return createColorStyle(Math.floor(r), Math.floor(g), Math.floor(b));
                }

                function Color(min) {
                    min = min || 0;
                    this.r = colorValue(min);
                    this.g = colorValue(min);
                    this.b = colorValue(min);
                    this.style = createColorStyle(this.r, this.g, this.b);
                }

                function Dot(){
                    this.x = Math.random() * canvas.width;
                    this.y = Math.random() * canvas.height;

                    this.vx = -.5 + Math.random();
                    this.vy = -.5 + Math.random();

                    this.radius = Math.random() * 2;

                    this.color = new Color();
                }

                Dot.prototype = {
                    draw: function(){
                        ctx.beginPath();
                        ctx.fillStyle = this.color.style;
                        ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2, false);
                        ctx.fill();
                    }
                };

                function createDots(){
                    for(let i = 0; i < dots.nb; i++){
                        dots.array.push(new Dot());
                    }
                }

                function moveDots() {
                    for(let i = 0; i < dots.nb; i++){

                        var dot = dots.array[i];

                        if(dot.y < 0 || dot.y > canvas.height){
                            dot.vx = dot.vx;
                            dot.vy = - dot.vy;
                        }
                        else if(dot.x < 0 || dot.x > canvas.width){
                            dot.vx = - dot.vx;
                            dot.vy = dot.vy;
                        }
                        dot.x += dot.vx;
                        dot.y += dot.vy;
                    }
                }

                function connectDots() {
                    for(let i = 0; i < dots.nb; i++){
                        for(let j = 0; j < dots.nb; j++){
                            let i_dot = dots.array[i];
                            let j_dot = dots.array[j];

                            if((i_dot.x - j_dot.x) < dots.distance && (i_dot.y - j_dot.y) < dots.distance && (i_dot.x - j_dot.x) > - dots.distance && (i_dot.y - j_dot.y) > - dots.distance){
                                if((i_dot.x - mousePosition.x) < dots.d_radius && (i_dot.y - mousePosition.y) < dots.d_radius && (i_dot.x - mousePosition.x) > - dots.d_radius && (i_dot.y - mousePosition.y) > - dots.d_radius){
                                    ctx.beginPath();
                                    ctx.strokeStyle = averageColorStyles(i_dot, j_dot);
                                    ctx.moveTo(i_dot.x, i_dot.y);
                                    ctx.lineTo(j_dot.x, j_dot.y);
                                    ctx.stroke();
                                    ctx.closePath();
                                }
                            }
                        }
                    }
                }

                function drawDots() {
                    for(let i = 0; i < dots.nb; i++){
                        var dot = dots.array[i];
                        dot.draw();
                    }
                }

                function animateDots() {
                    ctx.clearRect(0, 0, canvas.width, canvas.height);
                    moveDots();
                    connectDots();
                    drawDots();

                    requestAnimationFrame(animateDots);
                }

                canvas.onmousemove = function (e) {
                    mousePosition.x = e.pageX;
                    mousePosition.y = e.pageY;
                };

                canvas.mouseleave = function (e) {
                    mousePosition.x = canvas.width / 2;
                    mousePosition.y = canvas.height / 2;
                };

                createDots();
                requestAnimationFrame(animateDots);
            }
        }
    };
</script>
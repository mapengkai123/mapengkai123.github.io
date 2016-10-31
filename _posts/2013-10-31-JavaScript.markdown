---
layout: post
title:  "javascript表单验证"
date:   2013-10-31 20:10:38 +0800
---

<img src="/images/fulls/03.jpg" class="fit image"> 

	表单HTML
	<form action="" method="post">
	        <fieldset class="login">
	            <legend>Login information</legend>
	            <label for="userName" class="hover">UserName:</label>
	            <input type="text" id="userName" class="required text" /><br />
	            <label for="password" class="hover">Password:</label>
	            <input type="text" id="password" class="required text" /><br />
	        </fieldset>
	        <fieldset>
	            <legend>Personal Information</legend>
	            <label for="name" class="hover">Name:</label>
	            <input type="text" id="name" class="required text" /><br />
	            <label for="email" class="hover">Email:</label>
	            <input type="text" id="email" class="required text email" /><br />
	            <label for="date" class="hover">Date:</label>
	            <input type="text" id="date" class="required text" /><br />
	            <label for="url" class="hover">Website:</label>
	            <input type="text" id="url" class="url text" value="http://" /><br />
	            <label for="phone" class="hover">Phone:</label>
	            <input type="text" id="phone" class="phone text" /><br />
	            <label for="age" class="hover">Age:</label>
	            <input type="checkbox" id="age" name="age" value="yes" /><br />

	            <input type="submit" value="Submit Form" class="submit" />
	        </fieldset>
	    </form>


表单JavaScript

			var errMsg = {
            //检查特定字段是否必填
            required: {
                msg: 'This field is required.',
                test: function (obj, load) {
                    return obj.value.length > 0 && obj.value != obj.defaultValue;
                }
            },
            //确保字段内容是正确的email地址
            email: {
                msg: 'Not a valid email address.',
                test: function (obj) {//确保有内容的输入并符合email地址的格式
                    return !obj.value || /^[a-z0-9_+.-]+\@([a-z0-9-]+\.)+[a-z0-9]{2,4}$/i.test(obj.value);
                }
            },
            //确保字段内容是电话号码并将其自动格式化
            phone: {
                msg: 'Not a valid phone number',
                test: function (obj) {
                    var m = /(\d{3}).*(\d{3}).*(\d{4})/.exec(obj.value);
                    if (m) {
                        obj.value = "(" + m[1] + ")" + m[2] + "-" + m[3];
                        return !obj.value || m;
                    }
                }
            },
            //确保字段内容符合MM/DD/YYYY的时间格式
            date: {
                msg: 'Not a valid date.',
                test: function (obj) {
                    return !obj.value || /^\d{2}\/d{2}\/d{2,4}$/.test(obj.value);
                }
            },
            url: {
                msg: 'Not a valid URL.',
                test: function (obj) {
                    return !obj.value || obj.value == 'http:://' ||
                        /^https?:\/\/([a-z0-9-]+\.)+[a-z0-9]{2,4}.*$/.test(obj.value);
                }
            }

        };
        //隐藏当前正在显示的任何错误信息
        function hideErrors(elem) {
            var next = elem.nextSibling;//获取当前字段的下一个元素
            if (next && next.nodeName == 'UL' && next.className == 'errors') {//如果下一个元素是ul并有class为errors
                elem.parentNode.removeChild(next);//删掉它
            }
        }
        //显示表单内特定字段的错误信息
        function showErrors(elem, errors) {
            var next = elem.nextSibling;//获取当前字段的下一个元素
            if (next && (next.nodeName != 'UL' || next.className != 'errors')) {
                next = document.createElement('ul');
                next.className = 'errors';
                elem.parentNode.insertBefore(next, elem.nextSibling);
            }
            //有一个包含错误的容器引用，我们可以遍历所有的错误信息
            for (var i = 0; i < errors.length; i++) {
                var li = document.createElement('li');//为每一个错误信息创建新的li包裹器
                li.innerHTML = errors[i];
                next.appendChild(li);//并插入到dom中
            }
        }
        //验证表单所有字段的函数
        //form参数应是一个表单元素的引用 
        //load参数应该是一个布尔值  用以判别验证函数在页面加载时执行还是动态执行
        function validateForm(form, load) {
            var valid = true;
            for (var i = 0; i < form.elements.length; i++) {//遍历表单的所有字段的一个数组
                hideErrors(form.elements[i]);//先将所有错误信息隐藏
                if (!validateField(form.elements[i], load)) {
                    valid = false;
                }
            }
            return valid;//如果字段是不正确的内容，则返回false,反之则返回true;
        }
        //验证单个字段的内容
        function validateField(elem, load) {
            var errors = [];
            for (var name in errMsg) {
                var re = new RegExp("(^|\\s)" + name + "(\\s|$)");        
                if (re.test(elem.className) && !errMsg[name].test(elem, load)) {//如果没有通过验证，把错误信息增加到列表中
                    errors.push(errMsg[name].msg);
                }
            }
            if (errors.length) {//如果存在错误信息，则显示出来
                showErrors(elem, errors);
            }
            return errors.length > 0;
        }
       
        function getInputsByName(name) {
            var result = [];//匹配的input元素的数组
            result.numChecked = 0;//追踪被选中元素的数量
            var input = document.getElementsByTagName('input');
            for (var i = 0; i < input.length; i++) {
                if (input[i].name == name) {//找出所有指定name的字段
                    result.push(input[i]);//保存结果
                    if (input[i].checked) {//记录被选中字段的数量
                        result.numChecked++;
                    }
                }
            }
            return result;//返回匹配的字段集合
        }
        window.onload = function () {
            var form = document.getElementsByTagName('form')[0];
            form.onsubmit = function () {
                validateForm(form, 'submit');
                return false;
            }
        }

转自：静逸http://www.cnblogs.com/liyunhua/p/4526222.html
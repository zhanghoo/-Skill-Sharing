# 前端input图片的上传和压缩以及图片旋转90度问题解决
## 最近的微信项目中用到了input标签来上传图片文件，遇到两个大问题：

1、图片太大，导致上传速度慢，影响体验，需要压缩后上传；

2、有些手机拍照是横屏的，在页面展示的时候和正常方向相差90度，也就是本该立着的照片躺下了，需要根据具体情况修正后上传；

## 其实是拿手机拍照的方向问题，iphone正确的手机拍照方式是横屏的，用户往往是竖屏拍照等于照相机反转了９０度，出来的照片当然是反转９０度，当你横屏拍照上传，图片就是正确的，一张生成的图片是无法辨别选择方向的，只有在上传前反转角度才行，　因为上传到服务器以后，程序怎么可能知道这张照片要反转９０度，那张要反转１８０度，另一张要反转２７０度呢，其他的不用反转呢，正确的拍照姿势很重要呀，哈哈。。。蛋疼的问题

``` html

<div class="takeIn swiper-no-swiping" style="display: none;">
    <div class="takeIn-inner">
        <div class="pic-top">
            <img src="./images/myPics-top.png" alt="">
        </div>
        <div class="content-uploading">
            <div class="uploading">
                <div class="born-wrapper">
                    <input type="file" name="born" class="bornInp" id="bornInp">
                </div>
                <div class="vs-wrapper">
                    <img class="vs" src="./images/slide-1-vs.png" alt="">
                </div>
                <div class="beauty-wrapper">
                    <input type="file" name="beauty" class="beautyInp" id="beautyInp">
                </div>
            </div>
                    <textarea name="message" class="message" id="mes" cols="30" rows="2" maxlength="50"
                              placeholder="娃刚出生时，老公一度以为我是整容的，随着娃长大，我终于洗白了。"></textarea>
            <label class="check">
                <span id="check" class=""></span>
                <strong class="readyRead">本人承诺所上传图片符合法律法规，
                    并且拥有所有权或经所有人授权，因此造成的法律责任有本人承担</strong>
            </label>
            <button class="submit" id="submit" type="button">
                <img src="./images/submit.png" alt="">
            </button>
        </div>
    </div>
</div>

<script src="./js/exif.js" type="text/javascript"></script>// 用于获取图片的朝向信息Orientation 的插件

<script>
var $inpOutBorn = $(".born-wrapper");
var $inputBorn = $(".bornInp");
var $inpOutNew = $(".beauty-wrapper");
var $inputNew = $(".beautyInp");
var $readRule = $(".check");
var $submit = $("#submit");
var $successUpload = $(".successUpload");
var $message = $('#mes');
var $mareThanThree = $('.mareThanThree');
var $mention = $mareThanThree.find('.mention');
var check = false;
var Orientation;
var $downloadingPro = $('.downloadingPro');

var oldPic, newPic, matchingPic;// 存处理后的图片数据 提交用
var resultOld, resultNew;// 存图片地址 本地用

//函数调用，获取处理后的数据
upLoadPic($inputBorn, $inpOutBorn, 1);// 旧图片
upLoadPic($inputNew, $inpOutNew, 2);// 新图片

// 提示函数
function alertMention(mention) {
    $mareThanThree.show();
    $mention.html(mention);
}

// 阅读协议
$readRule.on('click', function () {
    $('#check').toggleClass('checked');
    check = !check;
});

// 图片处理
function upLoadPic(obj, out, num) {
    obj.change(function (e) {
        if (typeof FileReader != "undefined") {
            var reg = /(.jpg|.jpeg|.bmp|.png)$/;
            $($(this)[0].files).each(function () {
                var file = $(this);
                // console.log(typeof file[0].name.toLowerCase());
                if (reg.test(file[0].name.toLowerCase())) {
                    $downloadingPro.show();// 显示进度
                    var reader = new FileReader();

                    EXIF.getData($(this)[0], function () {/*获取图片方向信息*/
                        Orientation = EXIF.getTag(this, 'Orientation') || '';
                        console.log(Orientation)
                    });

                    reader.onload = function (e) {
                        var result = this.result;
                        var img = new Image();
                        img.src = result;
                        if (result.length < 200 * 1024) {
                            // 调用处理函数 小于200kb直接上传
                            if (num === 1) {
                                if (resultNew === result) {
                                    $downloadingPro.hide();
                                    alertMention('请上传不同的图片');
                                    return
                                }else if (Orientation === 3 || Orientation === 6 || Orientation === 8) {
                                    // console.log(Orientation)
                                    commonMethods.getImgData(result, Orientation, function (rotateData) {/*调用旋转函数*/
                                        // console.log(rotateData === result) false
                                        // console.log(rotateData === data) false
                                        out.css({
                                            'background': "url(" + rotateData + ") no-repeat center center",
                                            'background-size': 'contain'
                                        });
                                        oldPic = commonMethods.upload(rotateData, file[0].type);
                                    });
                                }else if (Orientation === '' || Orientation === 1) {
                                    out.css({
                                        'background': "url(" + result + ") no-repeat center center",
                                        'background-size': 'contain'
                                    });
                                    oldPic = commonMethods.upload(result, file[0].type);
                                }
                                resultOld = result;
                                //params.append("oldPic", file[0]);
                            } else {
                                if (result === resultOld) {
                                    $downloadingPro.hide();
                                    alertMention('请上传不同的图片');
                                    return
                                } else if (Orientation === 3 || Orientation === 6 || Orientation === 8) {
                                    commonMethods.getImgData(result, Orientation, function (rotateData) {/*7.15*/
                                        out.css({
                                            'background': "url(" + rotateData + ") no-repeat center center",
                                            'background-size': 'contain'
                                        });
                                        newPic = commonMethods.upload(rotateData, file[0].type);
                                    });
                                } else if (Orientation === '' || Orientation === 1) {
                                    out.css({
                                        'background': "url(" + result + ") no-repeat center center",
                                        'background-size': 'contain'
                                    });
                                    newPic = commonMethods.upload(result, file[0].type);
                                }
                                resultNew = result;
                            }
                            $downloadingPro.hide();
                            return
                        }
                        //超过200kb压缩 加载完后压缩
                        if (img.complete) {
                            callback();
                        } else {
                            img.onload = callback;
                        }
                        function callback() {
                            var data = commonMethods.compress(img);
                            // console.log(data) base64
                            if (data.length >= 3000000) {
                                $downloadingPro.hide();
                                alertMention('图片不得超过3M，请重新选择');
                                return
                            }
                            // 处理图片
                            if (num === 1) {
                                if (resultNew === result) {
                                    $downloadingPro.hide();
                                    alertMention('请上传不同的图片');
                                    return
                                } else if (Orientation === 3 || Orientation === 6 || Orientation === 8) {
                                    // console.log(Orientation)
                                    commonMethods.getImgData(result, Orientation, function (rotateData) {/*7.15*/
                                        // console.log(rotateData === result) false
                                        // console.log(rotateData === data) false
                                        out.css({
                                            'background': "url(" + rotateData + ") no-repeat center center",
                                            'background-size': 'contain'
                                        });
                                        oldPic = commonMethods.upload(rotateData, file[0].type);
                                    });
                                } else if (Orientation === '' || Orientation === 1) {
                                    out.css({
                                        'background': "url(" + result + ") no-repeat center center",
                                        'background-size': 'contain'
                                    });
                                    oldPic = commonMethods.upload(data, file[0].type);
                                }
                                resultOld = result;
                            } else if (num === 2) {
                                if (resultOld === result) {
                                    $downloadingPro.hide();
                                    alertMention('请上传不同的图片');
                                    return
                                } else if (Orientation === 3 || Orientation === 6 || Orientation === 8) {
                                    commonMethods.getImgData(result, Orientation, function (rotateData) {/*7.15*/
                                        out.css({
                                            'background': "url(" + rotateData + ") no-repeat center center",
                                            'background-size': 'contain'
                                        });
                                        newPic = commonMethods.upload(rotateData, file[0].type);
                                    });
                                } else if (Orientation === '' || Orientation === 1) {
                                    out.css({
                                        'background': "url(" + result + ") no-repeat center center",
                                        'background-size': 'contain'
                                    });
                                    newPic = commonMethods.upload(data, file[0].type);
                                }
                                resultNew = result;
                            }
                        }
                        $downloadingPro.hide();
                    }
                } else {
                    alert('只支持“jpg”、“jpeg”、“bmp”、“png”格式');
                    return
                }
                reader.readAsDataURL(file[0]);
            });
        }
    });
}
var commonMethods = {
    getImgData: function (img, dir, next) {
        var image = new Image();
        image.onload = function () {
            var degree = 0, drawWidth, drawHeight, width, height;
            drawWidth = this.naturalWidth;
            drawHeight = this.naturalHeight;
//以下改变一下图片大小
            var maxSide = Math.max(drawWidth, drawHeight);
            if (maxSide > 1024) {
                var minSide = Math.min(drawWidth, drawHeight);
                minSide = minSide / maxSide * 1024;
                maxSide = 1024;
                if (drawWidth > drawHeight) {
                    drawWidth = maxSide;
                    drawHeight = minSide;
                } else {
                    drawWidth = minSide;
                    drawHeight = maxSide;
                }
            }
            var canvas = document.createElement('canvas');
            canvas.width = width = drawWidth;
            canvas.height = height = drawHeight;
            var context = canvas.getContext('2d');
//判断图片方向，重置canvas大小，确定旋转角度，iphone默认的是home键在右方的横屏拍摄方式
            switch (dir) {
//iphone横屏拍摄，此时home键在左侧
                case 3:
                    degree = 180;
                    drawWidth = -width;
                    drawHeight = -height;
                    break;
//iphone竖屏拍摄，此时home键在下方(正常拿手机的方向)
                case 6:
                    canvas.width = height;
                    canvas.height = width;
                    degree = 90;
                    drawWidth = width;
                    drawHeight = -height;
                    break;
//iphone竖屏拍摄，此时home键在上方
                case 8:
                    canvas.width = height;
                    canvas.height = width;
                    degree = 270;
                    drawWidth = -width;
                    drawHeight = height;
                    break;
            }
//使用canvas旋转校正
            context.rotate(degree * Math.PI / 180);
            context.drawImage(this, 0, 0, drawWidth, drawHeight);
//返回校正图片
            next(canvas.toDataURL("image/jpeg", .8));
        }
        image.src = img;
    },
//  使用canvas对大图片进行压缩
    compress: function (img) {
        //    用于压缩图片的canvas
        var canvas = document.createElement("canvas");
        var ctx = canvas.getContext('2d');
        //    瓦片canvas
        var tCanvas = document.createElement("canvas");
        var tctx = tCanvas.getContext("2d");

        var initSize = img.src.length;
        var width = img.width;
        var height = img.height;
        //如果图片大于四百万像素，计算压缩比并将大小压至400万以下
        var ratio;
        if ((ratio = width * height / 4000000) > 1) {
            ratio = Math.sqrt(ratio);
            width /= ratio;
            height /= ratio;
        } else {
            ratio = 1;
        }
        canvas.width = width;
        canvas.height = height;
//        铺底色
        ctx.fillStyle = "#fff";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        //如果图片像素大于100万则使用瓦片绘制
        var count;
        if ((count = width * height / 1000000) > 1) {
            count = ~~(Math.sqrt(count) + 1); //计算要分成多少块瓦片
//            计算每块瓦片的宽和高
            var nw = ~~(width / count);
            var nh = ~~(height / count);
            tCanvas.width = nw;
            tCanvas.height = nh;
            for (var i = 0; i < count; i++) {
                for (var j = 0; j < count; j++) {
                    tctx.drawImage(img, i * nw * ratio, j * nh * ratio, nw * ratio, nh * ratio, 0, 0, nw, nh);
                    ctx.drawImage(tCanvas, i * nw, j * nh, nw, nh);
                }
            }
        } else {
            ctx.drawImage(img, 0, 0, width, height);
        }
        //进行最小压缩
        var ndata = canvas.toDataURL('image/jpeg', 0.1);
        console.log('压缩前：' + initSize);
        console.log('压缩后：' + ndata.length);
        console.log('压缩率：' + ~~(100 * (initSize - ndata.length) / initSize) + "%");
        tCanvas.width = tCanvas.height = canvas.width = canvas.height = 0;
        return ndata;
    },
    //图片转化 装入formdata里
    upload: function (basestr, type) {
        var text = window.atob(basestr.split(",")[1]);
        var buffer = new Uint8Array(text.length);
        for (var i = 0; i < text.length; i++) {
            buffer[i] = text.charCodeAt(i);
        }
        var data = this.getBlob([buffer], type);
        // var xhr = new XMLHttpRequest();
        var formdata = this.getFormData();
        formdata.append('imageFile', data);
        return formdata;
    },
    /**
     * 获取blob对象的兼容性写法
     * @param buffer
     * @param format
     * @returns {*}
     */
    getBlob: function (buffer, format) {
        try {
            return new Blob(buffer, {type: format});
        } catch (e) {
            var bb = new (window.BlobBuilder || window.WebKitBlobBuilder || window.MSBlobBuilder);
            buffer.forEach(function (buf) {
                bb.append(buf);
            });
            return bb.getBlob(format);
        }
    },
    /**
     * 获取formdata
     */
    getFormData: function () {
        var isNeedShim = ~navigator.userAgent.indexOf('Android')
            && ~navigator.vendor.indexOf('Google')
            && !~navigator.userAgent.indexOf('Chrome')
            && navigator.userAgent.match(/AppleWebKit\/(\d+)/).pop() <= 534;
        return isNeedShim ? new this.FormDataShim() : new FormData();
    },
    /**
     * formdata 补丁, 给不支持formdata上传blob的android机打补丁
     * @constructor
     */
    FormDataShim: function () {
        console.warn('using formdata shim');
        var o = this,
            parts = [],
            boundary = Array(21).join('-') + (+new Date() * (1e16 * Math.random())).toString(36),
            oldSend = XMLHttpRequest.prototype.send;
        this.append = function (name, value, filename) {
            parts.push('--' + boundary + '\r\nContent-Disposition: form-data; name="' + name + '"');
            if (value instanceof Blob) {
                parts.push('; filename="' + (filename || 'blob') + '"\r\nContent-Type: ' + value.type + '\r\n\r\n');
                parts.push(value);
            }
            else {
                parts.push('\r\n\r\n' + value);
            }
            parts.push('\r\n');
        };
        // Override XHR send()
        XMLHttpRequest.prototype.send = function (val) {
            var fr,
                data,
                oXHR = this;
            if (val === o) {
                // Append the final boundary string
                parts.push('--' + boundary + '--\r\n');
                // Create the blob
                data = this.getBlob(parts);
                // Set up and read the blob into an array to be sent
                fr = new FileReader();
                fr.onload = function () {
                    oldSend.call(oXHR, fr.result);
                };
                fr.onerror = function (err) {
                    throw err;
                };
                fr.readAsArrayBuffer(data);
                // Set the multipart content type and boudary
                this.setRequestHeader('Content-Type', 'multipart/form-data; boundary=' + boundary);
                XMLHttpRequest.prototype.send = oldSend;
            }
            else {
                oldSend.call(this, val);
            }
        };
    }
};
// 图片拼接
function picMatching(srcOld, srcNew) {
    var canvas = document.createElement('canvas');
    canvas.width = 200;
    canvas.height = 100;
    var ctx = canvas.getContext('2d');
    if (srcOld && srcNew) {
        var img1 = new Image();
        var img2 = new Image();
        var imgResult = new Image();
        img1.src = srcOld;
        img2.src = srcNew;
        for (var i = 0; i < 2; i++) {
            if (i === 0) {
                ctx.drawImage(img1, 0, 0, 100, 100);
            } else if (i === 1) {
                ctx.drawImage(img2, 100, 0, 100, 100);
            }
        }
        var srcMatching = canvas.toDataURL('image/png');
        imgResult.src = srcMatching;
        //转化为formData
        if (srcMatching.length < 3000000) {
            matchingPic = commonMethods.upload(srcMatching, 'image/png');// 拼接后的图片
            // console.log(matchingPic);
        } else {
            var data = commonMethods.compress(imgResult);// 压缩图片
            matchingPic = commonMethods.upload(data, 'image/png');
        }
        // $('.takeIn').append(imgResult);
        return matchingPic;
    }
}

// 最后提交数据
$submit.on('click', function () {
    var matchingData = picMatching(resultOld, resultNew);
    var message = $message.val();
    if (!(oldPic && newPic)) {
        alertMention('请选择图片'); // 如果没选择图片
        return
    } else if (!message) {
        alertMention('请填写留言'); // 如果没留言
        return
    } else if (!check) {
        alertMention('请勾选协议');// 如果没勾选协议
        return
    }
    if (oldPic && newPic && check && message) {
        $downloadingPro.show();// 显示进度 7.3
        //console.log(99)
        $.ajax({
            url: '',// 提交地址
            type: 'POST',
            data: {
                oldPic: oldPic,
                newPic: newPic,
                matchingPic: matchingData,
                message: message
            },
            /**
             * 必须false才会避开jQuery对 formdata 的默认处理
             * XMLHttpRequest会对 formdata 进行正确的处理
             */
            processData: false,
            /**
             *必须false才会自动加上正确的Content-Type
             */
            contentType: false
        }).done(function (res) {
            // 判断res值,上传成功 if(){$successUpload.show();} 删除进度条
            $('.balloon').hide();// 隐藏气球
            $downloadingPro.hide();// 7.3

            $successUpload.show();// 成功弹层
        }).fail(function (res) {
            alertMention('服务器繁忙，请稍后重试');//1111111111111111
        });
    }
});
</script>

```

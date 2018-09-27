

    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>敏感字查询</title>
        <script src="https://cdn.bootcss.com/jquery/1.12.4/jquery.min.js"></script>
        <script src="lodash.min.js"></script>
        <script src="badwords.js"></script>
    </head>
    <body>
        <input id="inputEmail" class="form-control" required autofocus>
        <button class="btn btn-lg btn-primary btn-block" type="button" id="test2">test</button>
        <script>
            var node = null;
            function translation(str) {
                var c = [];
                for (var i = 0; i < str.length; i++) {
                    var ch = str[i];
                    var chHex = str.charCodeAt(i).toString(16);
                    chHex = _.parseInt(chHex, 16);
                    /*全角=>半角*/
                    if (chHex > 0xFF00 && chHex < 0xFF5F)
                        chHex = (chHex - 0xFEE0);
                    /*大写=>小写*/
                    if (chHex > 0x40 && chHex < 0x5b)
                        chHex = (chHex + 0x20);
                    /*繁体=>简体*/
                    if (chHex > 0x4E00 && chHex < 0x9FFF) {
                        var index = traChn.indexOf(ch);
                        if (index > 0) {
                            ch = simChn[index];
                            chHex = simChn.charCodeAt(index).toString(16);
                            chHex = _.parseInt(chHex, 16);
                        }
                        ;
                    }
                    c.push(String.fromCharCode(chHex));
                }
                ;
                return c;
            };
            function isSkip(firstChar) {
                if (firstChar < '0')
                    return true;
                if (firstChar > '9' && firstChar < 'A')
                    return true;
                if (firstChar > 'Z' && firstChar < 'a')
                    return true;
                if (firstChar > 'z' && firstChar < 128)
                    return true;
                return false;
            };
            function find(src) {
                if (_.isNull(node)) {
                    throw '未初始化';
                }
                var result = [];
                if (_.isEmpty(src)) {
                    return result;
                };
                var text = translation(src)
                    , start = 0;
                while (start < text.length) {
                    var length = 0
                        , len = text.length - 1
                        , firstChar = text[start + length]
                        , curNode = {};
                    while (isSkip(firstChar) && (start + length) < len) {
                        start++;
                        firstChar = text[start + length];
                    }
                    while (!node[firstChar] && start < len) {
                        start++;
                        firstChar = text[start + length];
                    }
                    curNode = node;
                    while (typeof curNode[firstChar] !== 'undefined') {
                        curNode = curNode[firstChar];
                        length++;
                        if ((start + length) >= text.length) {
                            break;
                        }
                        ;
                        firstChar = text[start + length];
                        while (isSkip(firstChar) && !curNode.optEnd && (start + length) < len) {
                            length++;
                            firstChar = text[start + length];
                        }
                    }
                    if (curNode.isEnd || curNode.optEnd) {
                        result.push(src.substr(start, length));
                        start += length - 1;
                    }
                    ;
                    start++;
                }
                return result;
            }
            function init(keywords) {
                if (!_.isArray(keywords)) {
                    throw '关键词列表不是数组';
                }
                if (node !== null) {
                    return;
                }
                node = {};
                node.isEnd = false;
                _.forEach(keywords, function (keyword) {
                    var chs = translation(keyword);
                    var ch = chs[0];
                    var chNext = '';
                    if (typeof node[ch] === 'undefined') {
                        node[ch] = {};
                    }
                    var curNode = node[ch];
                    if (chs.length == 1) {
                        curNode.isEnd = true;
                        curNode.optEnd = true;
                    } else {
                        for (var i = 1; i < chs.length; i++) {
                            curNode.isEnd = false;
                            chNext = chs[i];
                            if (typeof curNode[chNext] === 'undefined') {
                                curNode[chNext] = {};
                            }
                            ;
                            if (i === chs.length - 1) {
                                curNode[chNext].isEnd = true;
                                curNode[chNext].optEnd = true;
                            }
                            ;
                            curNode = curNode[chNext];
                        }
                        ;
                    }
                })
            }
            //初始化库
            init(badwords);
            $("#test2").click(function () {
                var testwords = $("#inputEmail").val();
                console.log(testwords);
                var testresult = find(testwords);
                alert(testresult);
            })
        </script>
    </body>
    </html>
    
    
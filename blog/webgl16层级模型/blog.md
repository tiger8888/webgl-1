# 层级模型

> 层次模型是指用一颗“有向树”的数据结构来表示表示各类实体以及实体间的联系，树中每一个节点代表一个记录类型，树状结构表示实体型之间的联系。——百度百科

在变换时，父节点的变换会待着子节点一起，而子节点的变换则不会影响父节点。例如手臂的运动会带动手肘的运动，而手肘的运动不影响手臂的运动。即：

* 子节点的模型变换 = 父节点的模型变换 * 自身的模型变换

示例，如下两个立方体柱子，下面的为父节点，上面的未子节点。

```javascript
/**
 * 层次模型
 * xu.lidong@qq.com
 * */

g_RadParent = 0.0;
g_RadChild = 0.0;

function main() {
    var gl = getGL();
    var vsFile = "res/shader/pointlight.vert.glsl";
    var fsFile = "res/shader/pointlight.frag.glsl";

    initShaderProgram(gl, vsFile, fsFile, function (shaderProgram) {
        var n = initVertexBuffers(gl, shaderProgram);
        render(gl, shaderProgram, n);
        var speed = Math.PI * 0.02;
        document.onkeydown = function (ev) {
            switch (ev.keyCode) {
                case 37:// left
                    g_RadParent = (g_RadParent + speed) % (Math.PI * 2);
                    break;
                case 39:// right
                    g_RadParent = (g_RadParent - speed) % (Math.PI * 2);
                    break;
                case 38:// up
                    g_RadChild = (g_RadChild + speed) % (Math.PI * 2);
                    break;
                case 40:// down
                    g_RadChild = (g_RadChild - speed) % (Math.PI * 2);
                    break;
                default:
                    break;
            }
            render(gl, shaderProgram, n);
        }
    });
}

function render(gl, shaderProgram, n) {
    // 设置入射光
    var u_LightColor = gl.getUniformLocation(shaderProgram, "u_LightColor");
    gl.uniform3f(u_LightColor, 1.0, 1.0, 1.0);
    var u_LightPosition = gl.getUniformLocation(shaderProgram, "u_LightPosition");
    gl.uniform3f(u_LightPosition, 2.3, 4.0, 3.5);

    // 设置环境光
    var u_LightColorAmbient = gl.getUniformLocation(shaderProgram, "u_LightColorAmbient");
    gl.uniform3f(u_LightColorAmbient, 0.2, 0.2, 0.2);

    gl.clearColor(0.0, 0.0, 0.0, 1.0);
    gl.enable(gl.DEPTH_TEST);
    gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BYTE);

    // 父节点
    var u_ModelMatrix = gl.getUniformLocation(shaderProgram, "u_ModelMatrix");
    var transMatParent = getTranslationMatrix(0.0, -10, 0.0);
    var rotMatParent = getRotationMatrix(g_RadParent, 0, 1, 0);
    var modelMatParent = multiMatrix44(rotMatParent, transMatParent);
    gl.uniformMatrix4fv(u_ModelMatrix, false, modelMatParent);
    draw(gl, shaderProgram, n, modelMatParent);

    // 子节点
    var transMatChild = getTranslationMatrix(0.0, 10, 0.0);
    var rotMatChild = getRotationMatrix(g_RadChild, 0, 1, 0);
    var scaleMatChild = getScaleMatrix(0.8, 1.0, 0.8);
    var modelMatChild = multiMatrix44(transMatChild, modelMatParent);
    modelMatChild = multiMatrix44(rotMatChild, modelMatChild);
    modelMatChild = multiMatrix44(scaleMatChild, modelMatChild);
    gl.uniformMatrix4fv(u_ModelMatrix, false, modelMatChild);
    draw(gl, shaderProgram, n, modelMatChild);
}

function draw(gl, shaderProgram, n, modelMat) {
    // 逆转置矩阵
    var inverseMat = inverseMatrix(modelMat);
    var inverseTranposeMat = transposeMatrix(inverseMat);
    var u_NormalMatrix = gl.getUniformLocation(shaderProgram, "u_NormalMatrix");
    gl.uniformMatrix4fv(u_NormalMatrix, false, inverseTranposeMat);

    // mvp矩阵
    var viewMat = lookAt(20.0, 10.0, 30.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0);
    var projMat = getPerspectiveProjection(50, 16 / 9, 1, 100);
    var vpMat = multiMatrix44(projMat, viewMat);
    var mvpMat = multiMatrix44(vpMat, modelMat);
    var u_MvpMatrix = gl.getUniformLocation(shaderProgram, "u_MvpMatrix");
    gl.uniformMatrix4fv(u_MvpMatrix, false, mvpMat);

    gl.drawElements(gl.TRIANGLES, n, gl.UNSIGNED_BYTE, 0);
}

function initVertexBuffers(gl, sp) {
    var vertices = new Float32Array([
        1.5, 10.0, 1.5, -1.5, 10.0, 1.5, -1.5, 0.0, 1.5, 1.5, 0.0, 1.5,
        1.5, 10.0, 1.5, 1.5, 0.0, 1.5, 1.5, 0.0, -1.5, 1.5, 10.0, -1.5,
        1.5, 10.0, 1.5, 1.5, 10.0, -1.5, -1.5, 10.0, -1.5, -1.5, 10.0, 1.5,
        -1.5, 10.0, 1.5, -1.5, 10.0, -1.5, -1.5, 0.0, -1.5, -1.5, 0.0, 1.5,
        -1.5, 0.0, -1.5, 1.5, 0.0, -1.5, 1.5, 0.0, 1.5, -1.5, 0.0, 1.5,
        1.5, 0.0, -1.5, -1.5, 0.0, -1.5, -1.5, 10.0, -1.5, 1.5, 10.0, -1.5
    ]);

    var colors = new Float32Array([
        1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0,
        1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0,
        1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0,
        1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0,
        1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0,
        1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0
    ]);

    var normals = new Float32Array([
        0.0, 0.0, 1.0, 0.0, 0.0, 1.0, 0.0, 0.0, 1.0, 0.0, 0.0, 1.0,
        1.0, 0.0, 0.0, 1.0, 0.0, 0.0, 1.0, 0.0, 0.0, 1.0, 0.0, 0.0,
        0.0, 1.0, 0.0, 0.0, 1.0, 0.0, 0.0, 1.0, 0.0, 0.0, 1.0, 0.0,
        -1.0, 0.0, 0.0, -1.0, 0.0, 0.0, -1.0, 0.0, 0.0, -1.0, 0.0, 0.0,
        0.0, -1.0, 0.0, 0.0, -1.0, 0.0, 0.0, -1.0, 0.0, 0.0, -1.0, 0.0,
        0.0, 0.0, -1.0, 0.0, 0.0, -1.0, 0.0, 0.0, -1.0, 0.0, 0.0, -1.0
    ]);

    var indices = new Uint8Array([
        0, 1, 2, 0, 2, 3,
        4, 5, 6, 4, 6, 7,
        8, 9, 10, 8, 10, 11,
        12, 13, 14, 12, 14, 15,
        16, 17, 18, 16, 18, 19,
        20, 21, 22, 20, 22, 23
    ]);

    initArrayBuffer(gl, sp, vertices, 3, gl.FLOAT, "a_Position");
    initArrayBuffer(gl, sp, normals, 3, gl.FLOAT, "a_Normal");
    initArrayBuffer(gl, sp, colors, 3, gl.FLOAT, "a_Color");

    var ibo = gl.createBuffer();
    gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, ibo);
    gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, indices, gl.STATIC_DRAW);

    return indices.length;
}

function initArrayBuffer(gl, sp, data, num, type, attribute) {
    var buff = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, buff);
    gl.bufferData(gl.ARRAY_BUFFER, data, gl.STATIC_DRAW);

    var attr = gl.getAttribLocation(sp, attribute);
    gl.vertexAttribPointer(attr, num, type, false, 0, 0);
    gl.enableVertexAttribArray(attr);

    gl.bindBuffer(gl.ARRAY_BUFFER, null);
}

```

如果，当下方父节点转动时，会带动上方的子节点，子节点的运动，则不影响父节点。

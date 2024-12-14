# cocos2dx226_vc6
My cocos2d-x 2.2.6 vc6 study project, only for code study

## Avoid download file encode makes VC6 .dsp projects opening failed   
* unzip cocos2d-x-2.2.6.zip  

## Code reading, ccs2.txt    
```
cocos2d-x 2.2.6源码阅读感想，我这个是基于windows版的，不一定所有设备都是这样（视乎CCEGLView实现而不同）
（1）mainloop主循环外是CCApplication::run，循环内是CCDirector::drawScene（在glClear()后执行）
（2）循环外主要是glViewport，glLinkProgram和glUseProgram，初始化一次调用glLinkProgram
（3）循环内glClear和递归遍历显示列表中的所有对象的CCNode::visit()，
具体实现是逐个调用CCSprite::draw，它会依次调用glUseProgram，glVertexAttribPointer, glDrawArrays
```
```
_tWinMain
_int CCApplication::run(): modify this mainloop outside (while (1))
__bool AppDelegate::applicationDidFinishLaunching()
___void CCDirector::setOpenGLView(CCEGLView *pobOpenGLView)
____void CCDirector::setViewport()
_____void CCEGLView::setViewPortInPoints(float x , float y , float w , float h): glViewport
____bool CCGLProgram::link(): glLinkProgram
____void CCGLProgram::updateUniforms()->void CCGLProgram::use(): glUseProgram
__void CCDisplayLinkDirector::mainLoop(void)
<====================>enter mainloop 
___void CCDirector::drawScene(void): glClear, modify this mainloop inside (after glClear())
++++void CCNode::visit(): this->draw();
+++++void CCSprite::draw(void) enter use(): CC_NODE_DRAW_SETUP
++++++void CCGLProgram::use(): glUseProgram
+++++void CCSprite::draw(void) exit use(): (after use()) glVertexAttribPointer, glDrawArrays
****void CCEGLView::swapBuffers(): ::SwapBuffers
```
```
上次移植cocos2d-x 2.2.6到Miyoo A30失败（但3.6版则移植成功），查到原因了，
之所以黑屏完全是因为我移植的问题，我代码写得太急，没有在EGL初始化时（原版初始化GLFW）调用
bIsInit=true和CCEGLViewProtocol::setFrameSize，然而这两个操作极其重要，
决定了后面显示的点（可能会影响缩放），而且不会导致OpenGL报错
```
```
cocos2d-x debug info [OpenGL error 0x0502 in cocos2dx/sprite_nodes/CCSprite.cpp draw 584]
cocos2d-x debug info [OpenGL error 0x0502 in cocos2dx/sprite_nodes/CCSprite.cpp draw 584]
```

## Other projects  
* https://github.com/weimingtom/CrossApp_mingw  
* https://github.com/9miao/CrossApp  
* CrossApp_mingw.7z  

## Rotate cocos2d-x screen
* (not good) see cocos2dx/CCDirector.cpp, void CCDirector::drawScene(void), after kmGLPushMatrix();  
* (good) see cocos2dx/CCDirector.cpp, void CCDirector::setProjection(ccDirectorProjection kProjection), see
```
#if CC_TARGET_PLATFORM == CC_PLATFORM_WP8
            //if needed, we need to add a rotation for Landscape orientations on Windows Phone 8 since it is always in Portrait Mode
            kmGLMultMatrix(CCEGLView::sharedOpenGLView()->getOrientationMatrix());
#endif
```
* see cocos2dx/platform/wp8/CCEGLView.cpp, void CCEGLView::UpdateOrientationMatrix()
```
https://www.iteye.com/blog/weimingtom-1616972
https://cboard.cprogramming.com/game-programming/129635-opengl-es-2-0-how-implement-matrix-transforms-opengl-such-glloadidentity.html
https://webglfundamentals.org/webgl/lessons/webgl-2d-rotation.html
https://blog.csdn.net/ado137/article/details/37563323
https://docs.cocos2d-x.org/api-ref/cplusplus/V2.2.3/d1/da4/classcocos2d_1_1_c_c_scene.html
https://stackoverflow.com/questions/22374905/how-do-you-render-and-image-with-a-camera-perspective-in-opengl
https://blog.csdn.net/weixin_44053279/article/details/129713948
https://docs.cocos2d-x.org/api-ref/cplusplus/V2.2.3/d1/da4/classcocos2d_1_1_c_c_scene.html#ae3c0dadfbfae64c9dd1d5d9a6bec3d42
```
* 

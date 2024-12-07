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

安装GTK库 首先确保您的电脑已经安装了GTK库 执行和如下命令可以查看电脑上是否安装了GTK <br/>
#1、查看是否安装GTK
```
pkg-config --modversion gtk+ (查看1.2.x版本)
pkg-config --modversion gtk+-2.0 (查看 2.x 版本)
pkg-config --version (查看pkg-config的版本)
pkg-config --list-all grep gtk (查看是否安装了gtk)
```
2、如果没有安装，请安装GTK库 
或者执行如下命令安装GTK基本库就行<br/>
sudo apt-get install libgtk2.0-dev

3、编译动态库 
# cd ~ 
# vim sublime_imfix.c 内容如下： 
```
#include <gtk/gtkimcontext.h>

void 
gtk_im_context_set_client_window (
                                GtkIMContext *context,
                                GdkWindow    *window)
{
    GtkIMContextClass *klass;
    g_return_if_fail (GTK_IS_IM_CONTEXT (context));
    klass = GTK_IM_CONTEXT_GET_CLASS (context);

    if (klass->set_client_window)
    {
        klass->set_client_window (context, window);
    }
    g_object_set_data(G_OBJECT(context),"window",window);

    if(!GDK_IS_WINDOW (window))
    {
        return;
    }
    int width = gdk_window_get_width(window);
    int height = gdk_window_get_height(window);

    if(width != 0 && height !=0)
    {

    }

    gtk_im_context_focus_in(context);
}
```
:wq保存到～下

4、编译成共享库
```
gcc -shared -o libsublime-imfix.so sublime_imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC
```
拷贝到/opt/sublime_text目录下

sudo cp libsublime-imfix.so /opt/sublime_text/libsublime-imfix.so

注意：/opt/sublime_text/不同版本可能有所不同，请调整为自己安装版本的路径 
修改/usr/bin/subl文件，在第一行加入：

export LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so

5、修改sublime-text.desktop
sudo vim /usr/share/applications/sublime_text.desktop
```
[Desktop Entry]
Version=1.0
Type=Application
Name=Sublime Text
GenericName=Text Editor
Comment=Sophisticated text editor for code, markup and prose
#Exec=/opt/sublime_text/sublime_text %F 
Exec=/usr/bin/subl %F  #需要修改的地方
Terminal=false
MimeType=text/plain;
Icon=sublime-text
Categories=TextEditor;Development;Utility;
StartupNotify=true
Actions=Window;Document;

X-Desktop-File-Install-Version=0.22

[Desktop Action Window]
Name=New Window
#Exec=/opt/sublime_text/sublime_text -n  #需要修改的地方
Exec=/usr/bin/subl -n
OnlyShowIn=Unity;

[Desktop Action Document]
Name=New File
#Exec=/opt/sublime_text/sublime_text --command new_file
Exec=/usr/bin/subl new_file #需要修改的地方
OnlyShowIn=Unity;
```







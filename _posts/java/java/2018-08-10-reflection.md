---
title: "java - String(문자열)으로 Method(메소드) 호출 (reflection)"
categories: 
  - java
tags:
  - java
last_modified_at: 2018-08-10T10:27:00+09:00
author_profile: true
---
Java를 사용하다가 문뜩 String을 파라미터로 받아서 Method를 호출할 수 있는지 궁금해졌다.

### reflection
Java에서 [reflection](https://www.oracle.com/technical-resources/articles/java/javareflection.html)이란 런타임에 클래스, 메서드, 속성 등을 검사하고 동적으로 호출하는 java 기능이다.

reflectoin은 동일한 시스템 (또는 자체)에서 다른 코드를 검사 할 수있는 코드를 설명하는 데 사용된다.

## 예제

    {% highlight java %}
    import java.lang.reflect.Method;
    public class Test {

        public static void main(String[] args) {

            String TestClass = "ADSA"; // 원래는 패키지 경로까지 적어야 하나 같은 default 경로라 생략
            
            try {
                // 이것만 선언할 경우 static만 호출 => 여기서 System.out.println("난 static블럭에 있는 함수"); 호출
                Class testClass = Class.forName(TestClass); 

                Object newObj = testClass.newInstance(); // 클래스의 받아온 정보를 기반으로 객체를 생성 

                Method m = testClass.getMethod("show", boolean.class, Integer.class); // 메소드 불러오기 
                m.invoke(newObj, true, 3); // 메소드 호출 A 3 출력
                
                // testClass.getMethod("A").invoke(newObj);  A가 생성되었습니다. public일 때
                m = testClass.getDeclaredMethod("A"); 
                m.setAccessible(true); // private를 접근하기 위해서는 getDeclaredMethod 뒤에 해줘야 한다.
                m.invoke(newObj);   //  A가 생성되었습니다.

            } catch (Exception e) {
            }

        }
    }


    class ADSA {
        private void A() {
            System.out.println("A가 생성되었습니다.");
        }

        public void show(boolean showOK, Integer a) {
            if (showOK)
                System.out.println("A " + a + " 출력");
        }

        static {
            System.out.println("난 static블럭에 있는 함수");
        }
    }    
    {% endhighlight %}

**getDeclaredMethod** method의 정의를 들어가보면 다음과 같이 되어있다.

    {% highlight java %}
    @CallerSensitive
        public Method getDeclaredMethod(String name, Class... parameterTypes)
            throws NoSuchMethodException, SecurityException {
            checkMemberAccess(Member.DECLARED, Reflection.getCallerClass(), true);
            Method method = searchMethods(privateGetDeclaredMethods(false), name, parameterTypes);
            if (method == null) {
                throw new NoSuchMethodException(getName() + "." + name + argumentTypesToString(parameterTypes));
            }
            return method;
        }
    {% endhighlight %}    

이 메서드는 불러오려는 메서드가 **priavte 속성일 때 사용**하는 메서드이다.

하지만 불러올 뿐이지 사용을 하기 위해서는 `setAccessble`을 `true`로 설정해줘야 사용할 수 있다.



---
#### 참고 및 출처

reflection
- <https://www.oracle.com/technical-resources/articles/java/javareflection.html>
- <https://stackoverflow.com/questions/37628/what-is-reflection-and-why-is-it-useful>

- <https://stackoverflow.com/questions/160970/how-do-i-invoke-a-java-method-when-given-the-method-name-as-a-string>
- <http://sks3297.tistory.com/entry/JAVA-문자열로-클래스-만들기>
- <https://code.i-harness.com/ko/q/274ca>

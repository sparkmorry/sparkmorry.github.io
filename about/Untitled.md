The commom definition of closure is:
>A "closure" is a inner function that has access to variables in the outer function.

We can see it very clearly by debugging in chrome how the function flows:

1. var p = new Person();
2. function Person();
3. p.displayname = function(){
4. var disp = p.displayname();
5. alert(this); //object p
6. return function dis(){ 
7. disp();
8. alert(this);  //object window  }
9. }

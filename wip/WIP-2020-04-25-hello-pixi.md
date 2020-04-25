---
layout: page
title: "Hello PixiJS"
sub_title: "How to start in this?"
excerpt_separator: "<!--more-->"
categories:
  - PixiJS
elements:
  - openfl
  - haxe
  - pixijs
  - typescript
  - vscode
  - game development
---
Trying to get into Pixi fast isn't easy because the fast way of doing it is to drop raw javascript on your browser which isn't viable if you need to make something somewhat professional.  
On the other hand, javascript is a really complex world with scary words like "package manager", "bundler", "dependencies", "ecmascript" and more.

How do we enter this world?

<!--more-->

Let's start from the bottom:

# Haxe vs Typescript

## The elephant in the room
*Let's kill it!*  
Why not raw Javascript.

```js
0=="0" //true
0==[] //true
"0" == [] // false
[] == "0" // false (one guy once told me that this might be different. It wasn't)
```
meanwhile on typescript...
```ts
0 == "0" // ERROR: This condition will always return 'false' since the types 'number' and 'string' have no overlap. (TS2367)
```
Coming from C++, C#, and Haxe, strongly typing has become second nature for me. The desition for typescript was obvious.

## The target
The biggest difference between Haxe and Typescript is that the former is meant to transpile into a lot of different languages and platforms while Typescript only becomes Javascript.  
This is not a small feat, however, we were mostly making HTML5 games and when the time to put one of our games into native came, instead of using Haxe's native android/windows hxcpp export, we used Cordova and NWJS.

## Making the leap

Let's make a table
`var`
| Haxe | Typescript |
|---|---|
| `var` | `let` & `const`</br>*(`var` exists but it's evil and you shouldn't use it)* |
| `typedef` | `interface` |
| `for(e in arr)` | `for(e of arr)`</br>*(the `for in` version exists but does a different thing!)* |
| `switch` cases never fall through</br>*(the keyword `break` isn't needed)* | `switch` cases do fall through and you need to put your `break`!</br>*(see in the section below how to enforce the `break`)* |
| `Int` & `Float` | `number` |
| `Array<Thingy>` | `Thingy[]`  |
| `()->{}` | `()=>{}` |
| `new(){}` | `constructor(){}` |
| only `null` exists</br>*(anything that you don't set, is null)* | `null` is the null value and `undefined` means you didn't set it |
| everything by default is `private` | everything by default is `public` |
| overriding methods need the keyword `override` | you just declare the new method with the same name |

That is pretty much all there is to it! (I am lying of course, but with this, you should be ready to hit the ground running).

## Do I have to configure my language typescript?
This one caught me off guard the first time: You need to configure Typescript. Yes, you tell the language what you should and shouldn't be able to do and not the other way around, it's crazy!  
The file that does this is called `tsconfig.json` and tells the language where is your source code, where are your types and some rules about what you can and can't code.  
This is mine (json shouldn't have comments but this one can):
```json
{
  "compilerOptions": {
    "outDir": "./build/", // Where is my build going

    "strict": true, //enforces you to use types
    "strictPropertyInitialization": false, //forces you to assign a value to everything
    "strictNullChecks": false, //forces you to check that something is not null before using
    "noImplicitAny": true, //prevents the type guessing to be any
    "noImplicitReturns": true, //forces you to return what you promised you would return
    "noUnusedParameters": true, //Why would you have an unused parameter in your functions?
    "noUnusedLocals": true, //Why would you declare variables that you don't use?
    "noFallthroughCasesInSwitch": true, //This forces you to write break

    "sourceMap": true, //Helps debug
    "module": "ESNext", //Allows you to load the newest hip modules
    "moduleResolution": "node", //not sure, lol
    "allowSyntheticDefaultImports": true, //honestly no idea
    "experimentalDecorators": true, //something about incoming features

    //THIS IS IMPORTANT: What browser are you coding for? If you don't give a fuck about Internet Explorer set it to es6. otherwise you are stuck with es5 (and even then you might need polyfill).
    "target": "es5", 
    
    //Typescript can use javascript libraries as long as they have a d.ts file. Think of a .h but for javascript. This is where those files are.
    "typeRoots": [
      "node_modules/@types", //this is mandatory
      "src/types" //I like my types inside my src folder, but it can be anywhere.
    ]
  },

  //What files are part of your program? in my case, everything inside my src folder that ends in .ts
  "include": [
    "src/**/*.ts"
  ]
}
```


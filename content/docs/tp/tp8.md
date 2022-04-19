---
title: "Séance 8"
date: 2022-03-28T12:17:58+02:00
draft: false
weight: 8
---

# Séance 7

## Question 1
### Machine abstraite au début
```oz
[(
    declare A B C D
    thread D = C + 1 end
    thread C = B + 1 end
    thread A = 1 end
    thread B = A + 1 end
    {Browse D}
    ,
    {
        Browse->browse
    }
)], {browse = (...)}
```

### Machine abstraite après le declare
```oz
[(
    thread D = C + 1 end
    thread C = B + 1 end
    thread A = 1 end
    thread B = A + 1 end
    {Browse D}
    ,
    {
        Browse -> browse
        A->a
        B->b
        C->c
        D->d
    }
)], {browse = (...), a, b, c, d}
```

### Machine abstraite après la création des quatre threads
```oz
[(D = C + 1, {Browse -> browse, A -> a, B -> b, C -> c, D->d})]
[(C = B + 1, {Browse -> browse, A -> a, B -> b, C -> c, D->d})]
[(A = 1, {Browse -> browse, A -> a, B -> b, C -> c, D->d})]
[(B = A + 1, {Browse -> browse, A -> a, B -> b, C -> c, D->d})]
[(Browse D, {Browse -> browse, A -> a, B -> b, C -> c, D->d})]
, {browse = (...), a, b, c, d}
```

## Question 2
### Partie 1
#### Machine abstraite au début
```oz
[(
    local X Y Z in
        thread if X == 1 then Y = 2 else Z = 2 end
        thread if Y == 1 then X = 1 else Z = 2 end
        X = 1
    end
    ,
    {}
)], {}
```

#### Machine abstraite après une étape
```oz
[(
    thread if X == 1 then Y = 2 else Z = 2 end
    thread if Y == 1 then X = 1 else Z = 2 end
    X = 1
    ,
    {X -> x, Y -> y, Z -> z}
)], {x = _, y = _, z = _}
```

#### Machine abstraite après deux étapes
```oz
[(
    thread if X == 1 then Y = 2 else Z = 2 end
    ,
    {X -> x, Y -> y, Z -> z}
)]
[(
    thread if Y == 1 then X = 1 else Z = 2 end
    X = 1
    ,
    {X -> x, Y -> y, Z -> z}
)], {x, y, z}
```

### Partie 2
#### Premier code
```oz
[()],{x = 1, y = 2, z = 2}
```

#### Second code
```oz
[(if Y == 1 then X = 1 else Z = 2 end),{X -> x, Y -> y, Z -> z}],{x = 2, y, z = 2}
```

## Question 3
```oz
declare SumAux
fun {ProduceInts N} 
    fun {ProduceAux I N} 
        if I >= N then N|nil 
        else 
	        I|{ProduceAux I+1 N} 
        end 
    end 
in 
    {ProduceAux 1 N} 
end

fun {Sum Str}
    fun{SumAux Acc Str}
        case Str
        of nil then Acc
        [] H|T then {SumAux Acc+H T}
        end
    end
    {SumAux 0 Str}
end


declare Xs S
thread Xs = {ProduceInts 666} end
thread S = {Sum Xs} end
{Browse S}


%La première version prendra 666 secondes, la seconde deux fois plus longtemps
```

## Question 4
### Partie 1
```oz
declare 
fun {Producer N} 
    fun {ProduceAux I N} 
        if I >= N then N|nil 
        else 
	        I|{ProduceAux I+1 N} 
        end 
    end 
in 
    {ProduceAux 1 N} 
end 

fun {Filter L} 
    case L 
    of nil then nil 
    [] H|T then 
        if H mod 2 == 0 then {Filter T} 
        else H|{Filter T} 
        end 
    end 
end 

fun {Consumer L} 
    fun {SumAux L Acc} 
        case L 
        of nil then Acc 
        [] H|T then {SumAux T Acc+H} 
        end 
    end 
in 
    {SumAux L 0} 
end 

declare Xs Ys Zs 
thread Xs = {Producer 500} end
thread Ys = {Filter Xs} end 
thread Zs = {Consumer Ys} end 
{Browse Zs} 
```

### Partie 2
```oz
declare 

fun {Barman N} 
    {Delay 3000} 
    if N == 0 then nil 
    else 
         if {OS.rand} mod 2 == 1 then trappist|{Barman N-1} 
        else pils|{Barman N-1} 
        end 
    end 
end 

%hoih 
declare 
fun {Charlotte Str Acc Count} 
    fun {SmellTrappist Beer} 
        Beer == trappist 
    end 
in 
    case Str 
    of nil then Count = Acc nil 
    [] H|T then 
        if {SmellTrappist H} then 
	        {Charlotte T Acc+1 Count} 
        else 
	        H|{Charlotte T Acc Count} 
        end 
    end 
end 

declare 
fun {Friend Str Count} 
    case Str 
    of nil then Count 
    [] H|T then {Friend T Count+1} 
    end 
end 

local Bar Cha Fr C in 
thread Bar = {Barman 5} end 
thread Cha = {Charlotte Bar 0 C} end 
thread Fr = {Friend Cha 0} end 
{Browse C#Fr} 
end
```

## Question 5
```oz
declare 
fun {Counter Str}
    fun {UpdateL L Ch} 
        case L 
        of nil then (Ch#1)|nil 
        [] H|T then 
	        case H 
	        of H1#H2 then 
	            if H1 == Ch then 
	                (H1#(H2+1))|T 
	            else 
	                H|{UpdateL T Ch} 
	            end 
	        end 
        end 
    end 
    fun {CAcc S Acc}
        A2 
    in 
        case S 
        of H|T then 
            A2 = {UpdateL Acc H} 
            A2|{CAcc T A2} 
        else nil 
        end 
    end 
in 
    thread {CAcc Str nil} end 
end 

local InS in 
   {Browse {Counter InS}} 
   InS = a|b|a|c|d|e|b|e|_ 
end
```

## Question 6
```oz
declare 
fun {NotGate Xs} 
    fun {Not X} 
        1-X 
    end 
    fun {GateLoop F Xs} 
        case Xs 
            of X|Xr then {F X}|{GateLoop F Xr} 
            else nil 
        end 
    end 
in 
    thread {GateLoop Not Xs} end 
end 

fun {AndGate Xs Ys} 
    fun {And X Y} 
        X*Y 
    end 
    fun {GateLoop Xs Ys} 
        case Xs#Ys 
        of (X|Xr)#(Y|Yr) then {And X Y}|{AndGate Xr Yr} 
        end 
    end 
in 
    thread {GateLoop Xs Ys} end 
end 

fun {OrGate Xs Ys} 
    fun {Or X Y} 
        X+Y - X*Y 
    end 
    fun {GateLoop Xs Ys} 
        case Xs#Ys 
        of (X|Xr)#(Y|Yr) then {Or X Y}|{OrGate Xr Yr} 
        else nil 
        end 
    end 
in 
    thread {GateLoop Xs Ys} end 
end 
 
local S in 
    S = 0|1|0|0|1|0|nil
    Q = 1|0|0|0|1|0|0|_ 
    {Browse {OrGate S Q}} 
end 

declare 
fun {Simulate G Ss} 
    case G 
        of input(x) then Ss.x 
        [] input(y) then Ss.y 
        [] input(z) then Ss.z  
    else 
        case G.value 
        of 'or' then 
            thread {OrGate {Simulate G.1 Ss} {Simulate G.2 Ss}}end 
        [] 'and' then 
            thread {AndGate {Simulate G.1 Ss} {Simulate G.2 Ss}} end 
        [] 'not' then 
            thread {NotGate {Simulate G.1 Ss}} end 
        end 
    end 
end 

declare G Ss in 
    G = gate(value:'or' gate(value:'and' input(x) input(y)) gate(value:'not' input(z))) 
    {Browse {Simulate G Ss}} 
    Ss = input(x:1|0|1|0|_ y: 0|1|0|1|_ z:1|1|0|0|_)
end
```
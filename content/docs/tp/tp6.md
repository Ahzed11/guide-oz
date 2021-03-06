---
title: "Séance 6"
date: 2022-03-08T13:41:10+01:00
draft: false
weight: 6
---

# Séance 6

## Question 1
```oz
declare 
fun {Reverse L}
    C = {NewCell nil}
in
    for E in L do
        C := E|@C
    end
    @C
end

{Browse {Reverse [1 2 3 4 5]}}
```

## Question 2
```oz
declare 
fun {NewStack} 
    {NewCell nil} 
end 

fun {IsEmpty S} 
    @S == nil 
end 

proc {Push S X} 
   S := X|@S 
end 

fun {Pop S} 
    if {IsEmpty S} then nil 
    else R in 
        R = @S.1 
        S := @S.2 
        R 
    end 
end 

fun {Eval Xs} 
   S = {NewStack} 
in 
   for X in Xs do 
    case X 
        of '+' then {Push S {Pop S} + {Pop S}} 
        [] '-' then Op1 Op2 in 
	        Op1 = {Pop S} 
	        Op2 = {Pop S} 
	        {Push S Op2 - Op1} 
        [] '*' then {Push S {Pop S} * {Pop S}} 
        [] '/' then Op1 Op2 in 
	        Op1 = {Pop S} 
	        Op2 = {Pop S} 
	        {Push S Op2 div Op1} 
        [] N then {Push S N}
      end 
   end 
   {Pop S} 
end 

 

{Browse {Eval [13 45 '+' 89 17 '-' '*']}} % affiche 4176 = (13+45)*(89-17) 
{Browse {Eval [13 45 '+' 89 '*' 17 '-']}} % affiche 5145 = ((13+45)*89)-17 
{Browse {Eval [13 45 89 17 '-' '+' '*']}} % affiche 1521 
```

## Question 3
```oz
declare 
fun {NewStack}
    C = {NewCell nil}
    fun {IsEmpty} @C == nil end
    proc {Push X} C := X|@C end
    fun {Pop} if {IsEmpty} then nil else S in S=@C.1 C:=@C.2 S end end
in
    stack(isEmpty:IsEmpty push:Push pop:Pop)
end

declare 
fun {NewStack}
    C = {NewCell nil}
    fun {IsEmpty} @C == nil end
    proc {Push X} C := X|@C end
    fun {Pop} if {IsEmpty} then nil else S in S=@C.1 C:=@C.2 S end end
in
    stack(isEmpty:IsEmpty push:Push pop:Pop)
end

fun {Eval Xs} 
    S = {NewStack} 
 in 
    for X in Xs do 
        case X 
            of '+' then {S.push {S.pop}+{S.pop}} 
            [] '-' then Op1 Op2 in 
                Op1 = {S.pop}
                Op2 = {S.pop}
                {S.push Op2-Op1}
            [] '*' then {S.push {S.pop}*{S.pop}} 
            [] '/' then Op1 Op2 in 
                Op1 = {S.pop} 
                Op2 = {S.pop} 
                {S.push Op2 div Op1} 
            [] N then {S.push N}
       end 
    end 
    {S.pop} 
end 

{Browse {Eval [13 45 '+' 89 17 '-' '*']}} % affiche 4176 = (13+45)*(89-17) 
{Browse {Eval [13 45 '+' 89 '*' 17 '-']}} % affiche 5145 = ((13+45)*89)-17 
{Browse {Eval [13 45 89 17 '-' '+' '*']}} % affiche 1521 
```

## Question 4
```oz
declare 

fun {Shuffle L} I A B Li Picked in 
    I = {NewCell 0} 
    A = {NewArray 0 {Length L} 0} 
    for X in L do 
        A.@I := X 
        I := @I +1 
    end 
    Picked = {NewCell 0} 
    B = {NewArray 0 {Length L} 0} 
    for N in 0..({Length L}-1) do R in 
        R = {OS.rand} mod ({Length L} -N) 
        {Browse R} 
        Picked := A.R 
        B.N := @Picked 
        A.R := A.({Length L}-N-1) 
    end 
    Li = {NewCell nil} 
    for Ind in 0..({Length L}-1) do 
        Li :=(B.({Length L}-1-Ind))|@Li 
    end 
    @Li 
end 

 
{Browse {Shuffle [a b c d e]}}
```

## Question 5
L1 est une liste mais pas L2.

## Question 6
```oz
declare
class Collection
   attr elements
   meth init % initialise la collection
        elements:=nil
   end
   meth put(X) % insere X
        elements:=X|@elements
   end
   meth get($) % extrait un element et le renvoie
        case @elements of X|Xr then elements:=Xr X end
   end
   meth isEmpty($) % renvoie true ssi la collection est vide
        @elements==nil
   end
   meth union(C)
      if {Not {C isEmpty($)}} then
	    {self put({C get($)})}
	    {self union(C)}
      end
   end
   meth toList($)
      @elements
   end
end

declare
class SortedCollection from Collection
    meth put(X)
        fun {PutHelp L}
	        case L of H|T then
	            if H < X then H |{PutHelp T}
	            else X|L
	            end
	        [] nil then X|L
	        end
        end
    in
        elements := {PutHelp @elements}
   end
end

C1 = {New SortedCollection init}
C2 = {New Collection init}
{C1 put(0)}
{C1 put(1)}
{C2 put(2)}
{C2 put(1)}
{C2 put(3)}

{Browse {C1 toList($)}}
{Browse {C2 toList($)}}
{C1 union(C2)}
{Browse {C2 isEmpty($)}}
{Browse {C1 isEmpty($)}}
{Browse {C1 toList($)}}
{Browse {C1 get($)}}
{Browse {C1 toList($)}}

% if C1 , C2 are Collections , union is O (|C2|)
% because put is O (1)

% if C1 , C2 are SortedCollections ,
% union is O (|C2|^2 + |C1|*|C2|)

%proof :
%(n1 + (n1+1) + (n1+2) + ... + (n1+n2-1))
%~= (n1 + (n1+1) + (n1 +2) + ... + (n1+n2))
%= n2*n1 + (0+1+2+...+ n2)
% = n2*n1 + 0.5*n2*(n2+1)
% ~= n2*n1 + 0.5*n2*n2
% => O(n2^2 + n2*n1 )

declare
Xs = [7 8 0 4 3]
C3 = {New SortedCollection init}
for X in Xs do
   {C3 put(X)}
end
{Browse {C3 toList($)}}
% insertion sort O (n^2)
```

## Question 7
```oz
declare
class Constante
    attr value
    meth init(V) value := V end
    meth evalue($) @value end
    meth derive(V $) {New Constante init(0)} end
end
class Variable
    attr value
    meth init(V) value := V end
    meth evalue($) @value end
    meth derive(V E)
        if self == V then E = {New Constante init(1)}
        else E = {New Constante init(0)} end
    end
    meth set(V) value := V end
end
class Somme
    attr e1 e2
    meth init(Expr1 Expr2)
        e1 := Expr1
        e2 := Expr2
    end
    meth evalue($) {@e1 evalue($)} + {@e2 evalue($)} end
    meth derive(V $)
        {New Somme init({@e1 derive(V $)} {@e2 derive(V $)})}
    end
end
class Difference
    attr e1 e2
    meth init(Expr1 Expr2)
        e1 := Expr1
        e2 := Expr2
    end
    meth evalue($){@e1 evalue($)} - {@e2 evalue($)} end
    meth derive(V $)
        {New Difference init({@e1 derive(V $)} {@e2 derive(V $)}) }
   end
end
class Produit
    attr e1 e2
    meth init(Expr1 Expr2)
        e1 := Expr1
        e2 := Expr2
    end
    meth evalue($) {@e1 evalue($)} * {@e2 evalue($) } end
    meth derive(V $) D1 D2 in
        D1 = {New Produit init({@e1 derive(V $)} @e2)}
        D2 = {New Produit init({@e2 derive(V $) } @e1 ) }
        {New Somme init(D1 D2) }
    end
end
class Puissance
    attr e1 c
    meth init(Expr1 C)
        e1 := Expr1
        c := C
    end
    meth evalue($) {Pow {@e1 evalue($)}@c } end
    meth derive(V $) De1 P CP in
        De1 = {@e1 derive(V $)}
        P = {New Puissance init(@e1 @c -1)}
        CP ={New Produit init({ New Constante init(@c)} P)}
        {New Produit init(CP De1)}
   end
end

VarX ={New Variable init(0)}
VarY ={New Variable init(0)}
local
    ExprX2 ={New Puissance init(VarX 2) }
    Expr3 ={New Constante init(3) }
    Expr3X2 ={New Produit init(Expr3 ExprX2) }
    ExprXY ={New Produit init(VarX VarY) }
    Expr3X2mXY ={New Difference init(Expr3X2 ExprXY)}
    ExprY3 ={New Puissance init(VarY 3)}
in
    Formule ={New Somme init(Expr3X2mXY ExprY3)}
end

{VarX set(7)}
{VarY set(23)}
{Browse {Formule evalue($) }} % affiche 12153
{VarX set(5)}
{VarY set(8)}
{Browse {Formule evalue($)}} % affiche 547
declare
Derivee ={Formule derive(VarX $) } % represente 6 x - y
{VarX set(7)}
{VarY set(23)}
{Browse {Derivee evalue($)}} % affiche 19
```
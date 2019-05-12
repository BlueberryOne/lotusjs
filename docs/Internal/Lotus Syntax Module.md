# Lotus Syntax Module

*11th of May, 2019*

Last updated : *11th of May, 2019*

**Editors :**

- Blokyk

**Authors :**

- Blokyk

---

## Abstract

This document describes the basic structure and syntax of Lotus source code. It defines, in details, the tokenizing, parsing, and translation of Lotus.



## Overview

Lotus is an open-source language created to facilitate the work of web designers and web developpers. This document describes its syntax as well as the way it should be interpreted by a compiler to conform to the Lotus specification.



## Table of Contents

1. **[Introduction](#introduction)**
   
   - [x] [Visual and Writing Conventions](visual-and-writing-conventions)

2. **[Tokenizing and Parsing Lotus](tokenizing-and-parsing-lotus)**
   
   - [x] [Definitions](definitions)
   - [ ] [Token types](token-types)
   - [ ] [Tokens Railroad Diagrams](tokens-railroad-diagrams)
   - [ ] [Parser Railroad Diagrams](parser-railroad-diagrams)
   - [ ] [Parse Tree Model](parse-tree-model)
   - [ ] [Preprocessing the input stream](preprocessing-the-input-stream)

3. **[Tokenization](tokenization)**
   
   - [ ] [Consume a token](consume-a-token)
   - [ ] [Consume a string token](consume-a-string-token)
   - [ ] [Consume a number token](consume-a-number-token)
   - [ ] [Consume an escaped code point](consume-an-escaped-code-point)
   - [ ] [Consume a name](consume-a-name)
   - [ ] [Consume a ident-like token](consume-a-ident-like-token)
   - [ ] [Consume an identifier token](consume-an-identifier-token)
   - [ ] [Consume a function token](consume-a-function-token)
   - [ ] [Check if a code point would start an identifier](check-if-a-code-point-would-start-an-identifier)
   - [ ] [Check if two code points are a valid escape](check-if-two-code-points-are-a-valid-escape)
   - [ ] [Check if two code points would start a number](check-if-two-code-points-would-start-a-number)

4. **[Parsing](parsing)**
   
   - [ ] [Consume a list of tokens](consume-a-list-of-tokens)
   - [ ] [Consume a code block](consume-a-code-block)
   - [ ] [Consume an assignement](consume-an-assignement)
   - [ ] [Consume an operation](consume-an-operation)
   - [ ] [Consume a function call](consume-a-function-call)
   - [ ] [Consume a function declaration](consume-a-function-declaration)
   - [ ] [Consume an if clause](consume-an-if-clause)
   - [ ] [Consume an else clause](consume-an-else-clause)
   - [ ] [Consume a for loop declaration](consume-a-for-loop-declaration)
   - [ ] [Consume a foreach loop declaration](consume-a-foreach-loop-declaration)
   - [ ] [Consume a while loop declaration](consume-a-while-loop-declaration)
   - [ ] [Consume a do-while loop declaration](consume-a-do-while-loop-declaration)
   - [ ] [Consume a class declaration](consume-a-class-declaration)
   - [ ] [Consume an interface declaration](consume-an-interface-declaration)
   - [ ] [Consume an import directive](consume-an-import-directive)
   - [ ] [Consume a from directive](consume-a-from-directive)
   - [ ] [Consume a namespace directive](consume-a-namespace-directive)
   - [ ] [Consume an extends directive](consume-an-extends-directive)

## 1. Introduction

This document describes the syntax of Lotus source code.

It also defines algorithms to convert text into a list of tokens, and then transform it into an abstract syntax tree, that can be further interpreted and translated into HTML, CSS and Javascript.

### 1.1 Visual and Writing Conventions

**Notes :** Notes and informative sections are presented as a quote block and starts with the word "Note:".

> Note: This is an example of an informative section



**Examples :** Examples in this document are presented as a quote block and starts with the word "Example".

> Example : This is an example of an example section



**Short Code Segment :** Short code segments, such as token names, are presented in <code> tags.

`This is a short code segment`



**Multi-line Code Segment :** Long/Multi-line code segments, such as informal code examples, are presented in <pre> tags.

```
This is
A multiline 
Code segment
```



**Informal section :** Informal sections are preceded by the words "*This section is informative*" written in *italics*.



**Definition :** Words definitions are presented as h6 headers for the word(s) being defined, and the definition is presented in *italics* with a tab increment of 4 spaces.



## 2. Tokenizing and Parsing Lotus

*This section is informal*



Compiler and interpreter must use the parsing rules described in this specification to generate an *Abstract Syntax Tree* (AST) that can then be used to analyze, optimize or translate Lotus source code. 



### 2.1 Definitions

This section defines terms used throughout this 



###### code point

    A [Unicode code point](http://unicode.org/glossary/#code_point). A Unicode character with a value between 0 (hexadecimal) and 10FFFF (hexadecimal).

###### input stream

    A Unicode stream (or list) of [code points](code-point).

###### consume the next code point

    If the [input stream](input-stream) is treated as a stack, is analogous to poping a [code point](code-point) from the [input stream](input-stream) and returning the resulting [code point](code-point). If the [input stream](input-stream) is treated as an indexed list, is analogous to incrementing the list's index.

###### next input code point

    The first [code point](code-point) in the [input stream](input-stream) that has not been [consumed](consume-the-next-code-point) yet.

###### current input code point

    The last [code point](code-point) to have been [consumed](consume-the-next-code-point).

###### reconsume the current code point

    Push the [current input code point](current-input-code-point) on top of the [input stream](input-stream), so that next time you will need to [consume the next code point](consume-the-next-code-point), the [current input code point](current-input-code) will be [consumed](consume-the-next-code-point) instead.

###### EOF code point

    A conceptual [code point](code-point) used to represent the end of the [input stream](input-stream). If the [input stream](input-stream) is empty, the [next code point](next-code-point) is always an [EOF code point](eof-code-point).

###### digit

    A [code point](code-point) between U+0030 DIGIT ZERO (0) and U+0039 DIGIT NINE (9).

###### hex digit

    A [digit](digit), or a [code point](code-point) between U+0041 LATIN CAPITAL LETTER A (A) and U+0046 LATIN CAPITAL LETTER F (F), or a [code point](https://www.w3.org/TR/css-syntax-3/#code-point "code point") between U+0061 LATIN SMALL LETTER A (a) and U+0066 LATIN SMALL LETTER F (f).

###### uppercase letter

    A [code point](code-point) between U+0041 LATIN CAPITAL LETTER A (A) and U+005A LATIN CAPITAL LETTER Z (Z).

###### lowercase letter

    A [code point](code-point) between U+0061 LATIN SMALL LETTER A (a) and U+007A LATIN SMALL LETTER Z (z).

###### letter

    An [uppercase letter](uppercase-letter) or a [lowercase letter](lowercase-letter).

###### non-ASCII code point

    A [code point](code-point) with a value equal or greater than U+0080 <control>.

###### name code point

    A  [letter](letter), a [digit](digit), a [non-ASCII code point](non-ascii-code-point), a U+002D HYPHEN MINUS (-), or a U+005F LOW LINE (_).

###### newline

    A [code point](code-point) with a value of U+000A LINE FEED (represented as the escape sequence '\n' in most programming language).

###### whitespace

    A [newline](newline), a U+D800 CHARACTER TABULATION, or a U+0020 SPACE.

###### maximum allowed code point

    The [code point](code-point) with the value U+10FFF

###### input token stream

    The stream of tokens produced by the tokenizer.

###### current input token

    The last token from the [input token stream](input-token-stream) to have been [consumed](consume-a-token).

###### next input token

    The first token in the [input token stream](input-token-stream) that has not been [consumed](consume-a-token) yet.

###### EOF token

    A conceptual token used to represent the end of the [input token stream](input-token-stream). If the [input token stream](input-token-stream) is empty, the next token is always an [EOF token](eof-token).

###### reconsume the current input token

    Push the [current input token](current-input-token) on top of the [input token stream](input-token-stream), so that next time you will need to [consume the next token](consume-a-token), the [current input token](current-input-token) will be [consumed](consume-a-token) instead.



### 2.2 Token types


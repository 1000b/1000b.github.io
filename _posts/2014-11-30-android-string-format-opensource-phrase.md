---
layout: post
title: Android字符串格式化开源库phrase介绍
category: Android
tags: Android
keywords: Android,Phrase
description: ndroid字符串格式化开源库phrase介绍
---
### 引言
   [Phrase](https://github.com/square/phrase)是一个字符串格式化的开源项目，整个项目只有一个类Phrase.java，相比于String.format，通过phrase格式化字符串代码更具可读性。

### phrase项目介绍
- 源码：phrase项目的源代码很简单，里面总共只有一个类：Phrase.java，代码如下：
		
		package com.eebbk.learnspell.util;
		
		import java.util.HashMap;
		import java.util.HashSet;
		import java.util.Map;
		import java.util.Set;
		
		import android.app.Fragment;
		import android.content.Context;
		import android.content.res.Resources;
		import android.text.SpannableStringBuilder;
		import android.view.View;
		
		public final class Phrase {
		  private final CharSequence pattern;
		
		  private final Set<String> keys = new HashSet<String>();
		  private final Map<String, CharSequence> keysToValues = new HashMap<String, CharSequence>();
		
		  private CharSequence formatted;
		
		  private Token head;
		
		  private char curChar;
		  private int curCharIndex;
		
		  private static final int EOF = 0;
		
		  public static Phrase from(Fragment f, int patternResourceId) {
		    return from(f.getResources(), patternResourceId);
		  }
		
		  public static Phrase from(View v, int patternResourceId) {
		    return from(v.getResources(), patternResourceId);
		  }
		
		  public static Phrase from(Context c, int patternResourceId) {
		    return from(c.getResources(), patternResourceId);
		  }
		
		  public static Phrase from(Resources r, int patternResourceId) {
		    return from(r.getText(patternResourceId));
		  }
		
		  public static Phrase from(CharSequence pattern) {
		    return new Phrase(pattern);
		  }
		
		  public Phrase put(String key, CharSequence value) {
		    if (!keys.contains(key)) {
		      throw new IllegalArgumentException("Invalid key: " + key);
		    }
		    if (value == null) {
		      throw new IllegalArgumentException("Null value for '" + key + "'");
		    }
		    keysToValues.put(key, value);
		
		    formatted = null;
		    return this;
		  }
		
		  public Phrase put(String key, int value) {
		    if (!keys.contains(key)) {
		      throw new IllegalArgumentException("Invalid key: " + key);
		    }
		    keysToValues.put(key, Integer.toString(value));
		
		    formatted = null;
		    return this;
		  }
		
		  public Phrase putOptional(String key, CharSequence value) {
		    return keys.contains(key) ? put(key, value) : this;
		  }
		
		  public Phrase putOptional(String key, int value) {
		    return keys.contains(key) ? put(key, value) : this;
		  }
		
		  public CharSequence format() {
		    if (formatted == null) {
		      if (!keysToValues.keySet().containsAll(keys)) {
		        Set<String> missingKeys = new HashSet<String>(keys);
		        missingKeys.removeAll(keysToValues.keySet());
		        throw new IllegalArgumentException("Missing keys: " + missingKeys);
		      }
		
		      SpannableStringBuilder sb = new SpannableStringBuilder(pattern);
		      for (Token t = head; t != null; t = t.next) {
		        t.expand(sb, keysToValues);
		      }
		
		      formatted = sb;
		    }
		    return formatted;
		  }
		
		  @Override
		  public String toString() {
		    return pattern.toString();
		  }
		
		  private Phrase(CharSequence pattern) {
		    curChar = (pattern.length() > 0) ? pattern.charAt(0) : EOF;
		    this.pattern = pattern;
		
		    Token prev = null;
		    Token next;
		    while ((next = token(prev)) != null) {
		      if (head == null) head = next;
		      prev = next;
		    }
		  }
		
		  private Token token(Token prev) {
		    if (curChar == EOF) {
		      return null;
		    }
		    if (curChar == '{') {
		      char nextChar = lookahead();
		      if (nextChar == '{') {
		        return leftCurlyBracket(prev);
		      } else if (nextChar >= 'a' && nextChar <= 'z') {
		        return key(prev);
		      } else {
		        throw new IllegalArgumentException(
		            "Unexpected character '" + nextChar + "'; expected key.");
		      }
		    }
		    return text(prev);
		  }
		
		  private KeyToken key(Token prev) {
		    StringBuilder sb = new StringBuilder();
		
		    consume();
		    while ((curChar >= 'a' && curChar <= 'z') || curChar == '_') {
		      sb.append(curChar);
		      consume();
		    }
		
		    if (curChar != '}') {
		      throw new IllegalArgumentException("Missing closing brace: }");
		    }
		    consume();
		
		    if (sb.length() == 0) {
		      throw new IllegalArgumentException("Empty key: {}");
		    }
		
		    String key = sb.toString();
		    keys.add(key);
		    return new KeyToken(prev, key);
		  }
		
		  private TextToken text(Token prev) {
		    int startIndex = curCharIndex;
		
		    while (curChar != '{' && curChar != EOF) {
		      consume();
		    }
		    return new TextToken(prev, curCharIndex - startIndex);
		  }
		
		  private LeftCurlyBracketToken leftCurlyBracket(Token prev) {
		    consume();
		    consume();
		    return new LeftCurlyBracketToken(prev);
		  }
		
		  private char lookahead() {
		    return curCharIndex < pattern.length() - 1 ? pattern.charAt(curCharIndex + 1) : EOF;
		  }
		
		  private void consume() {
		    curCharIndex++;
		    curChar = (curCharIndex == pattern.length()) ? EOF : pattern.charAt(curCharIndex);
		  }
		
		  private abstract static class Token {
		    private final Token prev;
		    private Token next;
		
		    protected Token(Token prev) {
		      this.prev = prev;
		      if (prev != null) prev.next = this;
		    }
		
		    abstract void expand(SpannableStringBuilder target, Map<String, CharSequence> data);
		
		    abstract int getFormattedLength();
		
		    final int getFormattedStart() {
		      if (prev == null) {
		        return 0;
		      } else {
		        return prev.getFormattedStart() + prev.getFormattedLength();
		      }
		    }
		  }
		
		  private static class TextToken extends Token {
		    private final int textLength;
		
		    TextToken(Token prev, int textLength) {
		      super(prev);
		      this.textLength = textLength;
		    }
		
		    @Override void expand(SpannableStringBuilder target, Map<String, CharSequence> data) {
		    	
		    }
		
		    @Override int getFormattedLength() {
		      return textLength;
		    }
		  }
		
		  private static class LeftCurlyBracketToken extends Token {
		    LeftCurlyBracketToken(Token prev) {
		      super(prev);
		    }
		
		    @Override void expand(SpannableStringBuilder target, Map<String, CharSequence> data) {
		      int start = getFormattedStart();
		      target.replace(start, start + 2, "{");
		    }
		
		    @Override int getFormattedLength() {
		      return 1;
		    }
		  }
		
		  private static class KeyToken extends Token {
		    private final String key;
		
		    private CharSequence value;
		
		    KeyToken(Token prev, String key) {
		      super(prev);
		      this.key = key;
		    }
		
		    @Override void expand(SpannableStringBuilder target, Map<String, CharSequence> data) {
		      value = data.get(key);
		
		      int replaceFrom = getFormattedStart();
		      int replaceTo = replaceFrom + key.length() + 2;
		      target.replace(replaceFrom, replaceTo, value);
		    }
		
		    @Override int getFormattedLength() {
		      return value.length();
		    }
		  }
		}


- 字符串格式化原理  
通过阅读Phrase.java的代码可知，它用"{"和"}"将需要格式化的内容包起来，然后用键值对给需要改变的内容传值，包起来的内容为键，值为动态设置的内容，比如：

		"Hi {first_name}, you are {age} years old."  

我们要最终的显示内容为：“Hi UperOne, you are 26 years old.”这里的first_name和age是键，值为UperOne和26。

### 使用方法
使用起来很简单，通过键值对的方式插值：

		CharSequence parseStr = Phrase.from("Hi {first_name}, you are {age} years old.")
						 .put("first_name", "UperOne")
						 .put("age", "26")
						 .format();
				
		mParseTxt.setText( parseStr );
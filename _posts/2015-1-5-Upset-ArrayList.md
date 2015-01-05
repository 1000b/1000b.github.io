---
layout: post
title: 打乱ArrayList生成一个随机序列的序列
category: java
tags: java
keywords: java,ArrayList
description: 打乱ArrayList生成一个随机序列的序列
---

&emsp;&emsp;做试卷的时候，需要将一个句子中的单词、一个单词中的字符、选择题中的答题项打乱生成一个随机的序列，特将其抽象成工具类，方便复用。

	public static <V> boolean isEmpty(ArrayList<V> sourceList) {
        return (sourceList == null || sourceList.size() == 0);
    }

	/**
     * 打乱ArrayList
     * 
     * */
    public static <V> ArrayList<V> randomList(ArrayList<V> sourceList){
    	if (isEmpty(sourceList)) {
            return sourceList;
        }
    	
    	ArrayList<V> randomList = new ArrayList<V>( sourceList.size( ) );
    	do{
    		int randomIndex = Math.abs( new Random( ).nextInt( sourceList.size() ) );
        	randomList.add( sourceList.remove( randomIndex ) );
    	}while( sourceList.size( ) > 0 );
    	
    	return randomList;
    }
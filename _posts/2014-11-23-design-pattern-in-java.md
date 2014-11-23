---
layout: post
title: Java 类库中的设计模式
category: 工具
tags: Java
keywords: Java,Github
description: Java类库中的设计模式。
---
<p><b>本文摘自：<a href="http://stackoverflow.com/questions/1673841/examples-of-gof-design-patterns">Examples of GoF Design Patterns</a></b></p>  

<h3><a href="http://en.wikipedia.org/wiki/Abstract_factory_pattern">Abstract factory</a> <sup><sub>(recognizeable by creational methods returning the factory itself which in turn can be used to create another abstract/interface type)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javase/6/docs/api/javax/xml/parsers/DocumentBuilderFactory.html#newInstance%28%29"><code>javax.xml.parsers.DocumentBuilderFactory#newInstance()</code></a></li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/javax/xml/transform/TransformerFactory.html#newInstance%28%29"><code>javax.xml.transform.TransformerFactory#newInstance()</code></a></li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/javax/xml/xpath/XPathFactory.html#newInstance%28%29"><code>javax.xml.xpath.XPathFactory#newInstance()</code></a></li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Builder_pattern">Builder</a> <sup><sub>(recognizeable by creational methods returning the instance itself)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/lang/StringBuilder.html#append%28boolean%29"><code>java.lang.StringBuilder#append()</code></a> (unsynchronized)</li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/lang/StringBuffer.html#append%28boolean%29"><code>java.lang.StringBuffer#append()</code></a> (synchronized)</li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/nio/ByteBuffer.html#put%28byte%29"><code>java.nio.ByteBuffer#put()</code></a> (also on <a href="http://docs.oracle.com/javase/6/docs/api/java/nio/CharBuffer.html#put%28char%29"><code>CharBuffer</code></a>, <a href="http://docs.oracle.com/javase/6/docs/api/java/nio/ShortBuffer.html#put%28short%29"><code>ShortBuffer</code></a>, <a href="http://docs.oracle.com/javase/6/docs/api/java/nio/IntBuffer.html#put%28int%29"><code>IntBuffer</code></a>, <a href="http://docs.oracle.com/javase/6/docs/api/java/nio/LongBuffer.html#put%28long%29"><code>LongBuffer</code></a>, <a href="http://docs.oracle.com/javase/6/docs/api/java/nio/FloatBuffer.html#put%28float%29"><code>FloatBuffer</code></a> and <a href="http://docs.oracle.com/javase/6/docs/api/java/nio/DoubleBuffer.html#put%28double%29"><code>DoubleBuffer</code></a>)</li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/javax/swing/GroupLayout.Group.html#addComponent%28java.awt.Component%29"><code>javax.swing.GroupLayout.Group#addComponent()</code></a></li>
<li>All implementations of <a href="http://docs.oracle.com/javase/6/docs/api/java/lang/Appendable.html"><code>java.lang.Appendable</code></a></li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Factory_method_pattern">Factory method</a> <sup><sub>(recognizeable by creational methods returning an implementation of an abstract/interface type)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/util/Calendar.html#getInstance%28%29"><code>java.util.Calendar#getInstance()</code></a></li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/util/ResourceBundle.html#getBundle%28java.lang.String%29"><code>java.util.ResourceBundle#getBundle()</code></a></li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/text/NumberFormat.html#getInstance%28%29"><code>java.text.NumberFormat#getInstance()</code></a></li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/nio/charset/Charset.html#forName%28java.lang.String%29"><code>java.nio.charset.Charset#forName()</code></a></li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/net/URLStreamHandlerFactory.html"><code>java.net.URLStreamHandlerFactory#createURLStreamHandler(String)</code></a> (Returns singleton object per protocol)</li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Prototype_pattern">Prototype</a> <sup><sub>(recognizeable by creational methods returning a <em>different</em> instance of itself with the same properties)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/lang/Object.html#clone%28%29"><code>java.lang.Object#clone()</code></a> (the class has to implement <a href="http://docs.oracle.com/javase/6/docs/api/java/lang/Cloneable.html"><code>java.lang.Cloneable</code></a>)</li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Singleton_pattern">Singleton</a> <sup><sub>(recognizeable by creational methods returning the <em>same</em> instance (usually of itself) everytime)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/lang/Runtime.html#getRuntime%28%29"><code>java.lang.Runtime#getRuntime()</code></a></li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/awt/Desktop.html#getDesktop%28%29"><code>java.awt.Desktop#getDesktop()</code></a></li>
</ul>

<hr>

<h2><a href="http://en.wikipedia.org/wiki/Structural_pattern">Structural patterns</a></h2>

<h3><a href="http://en.wikipedia.org/wiki/Adapter_pattern">Adapter</a> <sup><sub>(recognizeable by creational methods taking an instance of <em>different</em> abstract/interface type and returning an implementation of own/another abstract/interface type which <em>decorates/overrides</em> the given instance)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/util/Arrays.html#asList%28T...%29"><code>java.util.Arrays#asList()</code></a></li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/io/InputStreamReader.html#InputStreamReader%28java.io.InputStream%29"><code>java.io.InputStreamReader(InputStream)</code></a> (returns a <code>Reader</code>)</li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/io/OutputStreamWriter.html#OutputStreamWriter%28java.io.OutputStream%29"><code>java.io.OutputStreamWriter(OutputStream)</code></a> (returns a <code>Writer</code>)</li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/javax/xml/bind/annotation/adapters/XmlAdapter.html#marshal%28BoundType%29"><code>javax.xml.bind.annotation.adapters.XmlAdapter#marshal()</code></a> and <a href="http://docs.oracle.com/javase/6/docs/api/javax/xml/bind/annotation/adapters/XmlAdapter.html#unmarshal%28ValueType%29"><code>#unmarshal()</code></a></li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Bridge_pattern">Bridge</a> <sup><sub>(recognizeable by creational methods taking an instance of <em>different</em> abstract/interface type and returning an implementation of own abstract/interface type which <em>delegates/uses</em> the given instance)</sub></sup></h3>

<ul>
<li>None comes to mind yet. A fictive example would be <code>new LinkedHashMap(LinkedHashSet&lt;K&gt;, List&lt;V&gt;)</code> which returns an unmodifiable linked map which doesn't clone the items, but <em>uses</em> them. The <a href="http://docs.oracle.com/javase/6/docs/api/java/util/Collections.html#newSetFromMap%28java.util.Map%29"><code>java.util.Collections#newSetFromMap()</code></a> and <a href="http://docs.oracle.com/javase/6/docs/api/java/util/Collections.html#singleton%28T%29"><code>singletonXXX()</code></a> methods however comes close.</li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Composite_pattern">Composite</a> <sup><sub>(recognizeable by behavioral methods taking an instance of <em>same</em> abstract/interface type into a tree structure)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/awt/Container.html#add%28java.awt.Component%29"><code>java.awt.Container#add(Component)</code></a> (practically all over Swing thus)</li>
<li><a href="http://docs.oracle.com/javaee/6/api/javax/faces/component/UIComponent.html#getChildren%28%29"><code>javax.faces.component.UIComponent#getChildren()</code></a> (practically all over JSF UI thus)</li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Decorator_pattern">Decorator</a> <sup><sub>(recognizeable by creational methods taking an instance of <em>same</em> abstract/interface type which adds additional behaviour)</sub></sup></h3>

<ul>
<li>All subclasses of <a href="http://docs.oracle.com/javase/6/docs/api/java/io/InputStream.html"><code>java.io.InputStream</code></a>, <a href="http://docs.oracle.com/javase/6/docs/api/java/io/OutputStream.html"><code>OutputStream</code></a>, <a href="http://docs.oracle.com/javase/6/docs/api/java/io/Reader.html"><code>Reader</code></a> and <a href="http://docs.oracle.com/javase/6/docs/api/java/io/Writer.html"><code>Writer</code></a> have a constructor taking an instance of same type.</li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/util/Collections.html"><code>java.util.Collections</code></a>, the <a href="http://docs.oracle.com/javase/6/docs/api/java/util/Collections.html#checkedCollection%28java.util.Collection,%20java.lang.Class%29"><code>checkedXXX()</code></a>, <a href="http://docs.oracle.com/javase/6/docs/api/java/util/Collections.html#synchronizedCollection%28java.util.Collection%29"><code>synchronizedXXX()</code></a> and <a href="http://docs.oracle.com/javase/6/docs/api/java/util/Collections.html#unmodifiableCollection%28java.util.Collection%29"><code>unmodifiableXXX()</code></a> methods.</li>
<li><a href="http://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequestWrapper.html"><code>javax.servlet.http.HttpServletRequestWrapper</code></a> and <a href="http://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletResponseWrapper.html"><code>HttpServletResponseWrapper</code></a></li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Facade_pattern">Facade</a> <sup><sub>(recognizeable by behavioral methods which internally uses instances of <em>different</em> independent abstract/interface types)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javaee/6/api/javax/faces/context/FacesContext.html"><code>javax.faces.context.FacesContext</code></a>, it internally uses among others the abstract/interface types <a href="http://docs.oracle.com/javaee/6/api/javax/faces/lifecycle/Lifecycle.html"><code>LifeCycle</code></a>, <a href="http://docs.oracle.com/javaee/6/api/javax/faces/application/ViewHandler.html"><code>ViewHandler</code></a>, <a href="http://docs.oracle.com/javaee/6/api/javax/faces/application/NavigationHandler.html"><code>NavigationHandler</code></a> and many more without that the enduser has to worry about it (which are however overrideable by injection).</li>
<li><a href="http://docs.oracle.com/javaee/6/api/javax/faces/context/ExternalContext.html"><code>javax.faces.context.ExternalContext</code></a>, which internally uses <a href="http://docs.oracle.com/javaee/6/api/javax/servlet/ServletContext.html"><code>ServletContext</code></a>, <a href="http://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpSession.html"><code>HttpSession</code></a>, <a href="http://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html"><code>HttpServletRequest</code></a>, <a href="http://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletResponse.html"><code>HttpServletResponse</code></a>, etc.</li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Flyweight_pattern">Flyweight</a> <sup><sub>(recognizeable by creational methods returning a cached instance, a bit the "multiton" idea)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/lang/Integer.html#valueOf%28int%29"><code>java.lang.Integer#valueOf(int)</code></a> (also on <a href="http://docs.oracle.com/javase/6/docs/api/java/lang/Boolean.html#valueOf%28boolean%29"><code>Boolean</code></a>, <a href="http://docs.oracle.com/javase/6/docs/api/java/lang/Byte.html#valueOf%28byte%29"><code>Byte</code></a>, <a href="http://docs.oracle.com/javase/6/docs/api/java/lang/Character.html#valueOf%28char%29"><code>Character</code></a>, <a href="http://docs.oracle.com/javase/6/docs/api/java/lang/Short.html#valueOf%28short%29"><code>Short</code></a> and <a href="http://docs.oracle.com/javase/6/docs/api/java/lang/Long.html#valueOf%28long%29"><code>Long</code></a>)</li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Proxy_pattern">Proxy</a> <sup><sub>(recognizeable by creational methods which returns an implementation of given abstract/interface type which in turn <em>delegates/uses</em> a <em>different</em> implementation of given abstract/interface type)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/lang/reflect/Proxy.html"><code>java.lang.reflect.Proxy</code></a></li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/rmi/package-summary.html"><code>java.rmi.*</code></a>, the whole API actually.</li>
</ul>

<p><sup><sub>The Wikipedia example is IMHO a bit poor, lazy loading has actually completely nothing to do with the proxy pattern at all.</sub></sup></p>

<hr>

<h2><a href="http://en.wikipedia.org/wiki/Behavioral_pattern">Behavioral patterns</a></h2>

<h3><a href="http://en.wikipedia.org/wiki/Chain_of_responsibility_pattern">Chain of responsibility</a> <sup><sub>(recognizeable by behavioral methods which (indirectly) invokes the same method in <em>another</em> implementation of <em>same</em> abstract/interface type in a queue)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/util/logging/Logger.html#log%28java.util.logging.Level,%20java.lang.String%29"><code>java.util.logging.Logger#log()</code></a></li>
<li><a href="http://docs.oracle.com/javaee/6/api/javax/servlet/Filter.html#doFilter%28javax.servlet.ServletRequest,%20javax.servlet.ServletResponse,%20javax.servlet.FilterChain%29"><code>javax.servlet.Filter#doFilter()</code></a></li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Command_pattern">Command</a> <sup><sub>(recognizeable by behavioral methods in an abstract/interface type which invokes a method in an implementation of a <em>different</em> abstract/interface type which has been <em>encapsulated</em> by the command implementation during its creation)</sub></sup></h3>

<ul>
<li>All implementations of <a href="http://docs.oracle.com/javase/6/docs/api/java/lang/Runnable.html"><code>java.lang.Runnable</code></a></li>
<li>All implementations of <a href="http://docs.oracle.com/javase/6/docs/api/javax/swing/Action.html"><code>javax.swing.Action</code></a></li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Interpreter_pattern">Interpreter</a> <sup><sub>(recognizeable by behavioral methods returning a <em>structurally</em> different instance/type of the given instance/type; note that parsing/formatting is not part of the pattern, determining the pattern and how to apply it is)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/util/regex/Pattern.html"><code>java.util.Pattern</code></a></li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/text/Normalizer.html"><code>java.text.Normalizer</code></a></li>
<li>All subclasses of <a href="http://docs.oracle.com/javase/6/docs/api/java/text/Format.html"><code>java.text.Format</code></a></li>
<li>All subclasses of <a href="http://docs.oracle.com/javaee/6/api/javax/el/ELResolver.html"><code>javax.el.ELResolver</code></a></li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Iterator_pattern">Iterator</a> <sup><sub>(recognizeable by behavioral methods sequentially returning instances of a <em>different</em> type from a queue)</sub></sup></h3>

<ul>
<li>All implementations of <a href="http://docs.oracle.com/javase/6/docs/api/java/util/Iterator.html"><code>java.util.Iterator</code></a> (thus among others also <a href="http://docs.oracle.com/javase/6/docs/api/java/util/Scanner.html"><code>java.util.Scanner</code></a>!).</li>
<li>All implementations of <a href="http://docs.oracle.com/javase/6/docs/api/java/util/Enumeration.html"><code>java.util.Enumeration</code></a></li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Mediator_pattern">Mediator</a> <sup><sub>(recognizeable by behavioral methods taking an instance of different abstract/interface type (usually using the command pattern) which delegates/uses the given instance)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/util/Timer.html"><code>java.util.Timer</code></a> (all <code>scheduleXXX()</code> methods)</li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/Executor.html#execute%28java.lang.Runnable%29"><code>java.util.concurrent.Executor#execute()</code></a></li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/ExecutorService.html"><code>java.util.concurrent.ExecutorService</code></a> (the <code>invokeXXX()</code> and <code>submit()</code> methods)</li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/ScheduledExecutorService.html"><code>java.util.concurrent.ScheduledExecutorService</code></a> (all <code>scheduleXXX()</code> methods)</li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/lang/reflect/Method.html#invoke%28java.lang.Object,%20java.lang.Object...%29"><code>java.lang.reflect.Method#invoke()</code></a></li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Memento_pattern">Memento</a> <sup><sub>(recognizeable by behavioral methods which internally changes the state of the <em>whole</em> instance)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/util/Date.html"><code>java.util.Date</code></a> (the setter methods do that, <code>Date</code> is internally represented by a <code>long</code> value)</li>
<li>All implementations of <a href="http://docs.oracle.com/javase/6/docs/api/java/io/Serializable.html"><code>java.io.Serializable</code></a></li>
<li>All implementations of <a href="http://docs.oracle.com/javaee/6/api/javax/faces/component/StateHolder.html"><code>javax.faces.component.StateHolder</code></a></li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Observer_pattern">Observer (or Publish/Subscribe)</a> <sup><sub>(recognizeable by behavioral methods which invokes a method on an instance of <em>another</em> abstract/interface type, depending on own state)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/util/Observer.html"><code>java.util.Observer</code></a>/<a href="http://docs.oracle.com/javase/6/docs/api/java/util/Observable.html"><code>java.util.Observable</code></a> (rarely used in real world though)</li>
<li>All implementations of <a href="http://docs.oracle.com/javase/6/docs/api/java/util/EventListener.html"><code>java.util.EventListener</code></a> (practically all over Swing thus)</li>
<li><a href="http://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpSessionBindingListener.html"><code>javax.servlet.http.HttpSessionBindingListener</code></a></li>
<li><a href="http://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpSessionAttributeListener.html"><code>javax.servlet.http.HttpSessionAttributeListener</code></a></li>
<li><a href="http://docs.oracle.com/javaee/6/api/javax/faces/event/PhaseListener.html"><code>javax.faces.event.PhaseListener</code></a></li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/State_pattern">State</a> <sup><sub>(recognizeable by behavioral methods which changes its behaviour depending on the instance's state which can be controlled externally)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javaee/6/api/javax/faces/lifecycle/Lifecycle.html#execute%28javax.faces.context.FacesContext%29"><code>javax.faces.lifecycle.LifeCycle#execute()</code></a> (controlled by <a href="http://docs.oracle.com/javaee/6/api/javax/faces/webapp/FacesServlet.html"><code>FacesServlet</code></a>, the behaviour is dependent on current phase (state) of JSF lifecycle)</li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Strategy_pattern">Strategy</a> <sup><sub>(recognizeable by behavioral methods in an abstract/interface type which invokes a method in an implementation of a <em>different</em> abstract/interface type which has been <em>passed-in</em> as method argument into the strategy implementation)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javase/6/docs/api/java/util/Comparator.html#compare%28T,%20T%29"><code>java.util.Comparator#compare()</code></a>, executed by among others <code>Collections#sort()</code>.</li>
<li><a href="http://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServlet.html"><code>javax.servlet.http.HttpServlet</code></a>, the <code>service()</code> and all <code>doXXX()</code> methods take <code>HttpServletRequest</code> and <code>HttpServletResponse</code> and the implementor has to process them (and not to get hold of them as instance variables!).</li>
<li><a href="http://docs.oracle.com/javaee/6/api/javax/servlet/Filter.html#doFilter%28javax.servlet.ServletRequest,%20javax.servlet.ServletResponse,%20javax.servlet.FilterChain%29"><code>javax.servlet.Filter#doFilter()</code></a></li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Template_method_pattern">Template method</a> <sup><sub>(recognizeable by behavioral methods which already have a "default" behaviour definied by an abstract type)</sub></sup></h3>

<ul>
<li>All non-abstract methods of <a href="http://docs.oracle.com/javase/6/docs/api/java/io/InputStream.html"><code>java.io.InputStream</code></a>, <a href="http://docs.oracle.com/javase/6/docs/api/java/io/OutputStream.html"><code>java.io.OutputStream</code></a>, <a href="http://docs.oracle.com/javase/6/docs/api/java/io/Reader.html"><code>java.io.Reader</code></a> and <a href="http://docs.oracle.com/javase/6/docs/api/java/io/Writer.html"><code>java.io.Writer</code></a>.</li>
<li>All non-abstract methods of <a href="http://docs.oracle.com/javase/6/docs/api/java/util/AbstractList.html"><code>java.util.AbstractList</code></a>, <a href="http://docs.oracle.com/javase/6/docs/api/java/util/AbstractSet.html"><code>java.util.AbstractSet</code></a> and <a href="http://docs.oracle.com/javase/6/docs/api/java/util/AbstractMap.html"><code>java.util.AbstractMap</code></a>.</li>
<li><a href="http://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServlet.html"><code>javax.servlet.http.HttpServlet</code></a>, all the <code>doXXX()</code> methods by default sends a HTTP 405 "Method Not Allowed" error to the response. You're free to implement none or any of them.</li>
</ul>

<h3><a href="http://en.wikipedia.org/wiki/Visitor_pattern">Visitor</a> <sup><sub>(recognizeable by two <em>different</em> abstract/interface types which has methods definied which takes each the <em>other</em> abstract/interface type; the one actually calls the method of the other and the other executes the desired strategy on it)</sub></sup></h3>

<ul>
<li><a href="http://docs.oracle.com/javase/6/docs/api/javax/lang/model/element/AnnotationValue.html"><code>javax.lang.model.element.AnnotationValue</code></a> and <a href="http://docs.oracle.com/javase/6/docs/api/javax/lang/model/element/AnnotationValueVisitor.html"><code>AnnotationValueVisitor</code></a></li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/javax/lang/model/element/Element.html"><code>javax.lang.model.element.Element</code></a> and <a href="http://docs.oracle.com/javase/6/docs/api/javax/lang/model/element/ElementVisitor.html"><code>ElementVisitor</code></a></li>
<li><a href="http://docs.oracle.com/javase/6/docs/api/javax/lang/model/type/TypeMirror.html"><code>javax.lang.model.type.TypeMirror</code></a> and <a href="http://docs.oracle.com/javase/6/docs/api/javax/lang/model/type/TypeVisitor.html"><code>TypeVisitor</code></a></li>
</ul>
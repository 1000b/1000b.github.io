---
layout: post
title: Android EditText的使用及值得注意的地方
category: Android
tags: Android
keywords: Android,EditText
description: Android EditText的使用及值得注意的地方
---

&emsp;&emsp;Android上有很多输入法应用，每种输入法都有各自的特点，输入法多数时候是和EditText配合使用，结合我自己的亲身实践分享一下使用EditText过程中遇到的一些问题及解决方法。

### 设置默认输入法

&emsp;&emsp;有时候为了提高用户体验，在弹出输入法时需要设置默认的输入状态，比如单词应用弹出输入法时，输入法最好是在英文输入状态下。如果是字典应用，弹出输入法时最好是在中文输入状态下，Android并没有提供设置默认的输入状态的接口，但我们可以通过如下方法一样能够达到想要的效果：

&emsp;&emsp;默认中文：

	mEditText.setInputType(EditorInfo.TYPE_CLASS_TEXT);

&emsp;&emsp;默认英文：

	mEditText.setInputType(EditorInfo.TYPE_TEXT_VARIATION_URI);

### 打开和关闭输入法

&emsp;&emsp;手动控制输入法的开关状态也能提升用户体验，比如：

- 有的搜索框会有一个清除按钮，点击清除按钮时就应该弹出输入法，因为用户清除搜索内容的目的多数时候是需要输入新的内容；

- 执行搜索时应该隐藏输入法，因为显示输入法时会遮挡搜索结果，用户体验不太好；

- 闹钟来时或者有其它window弹出时应该隐藏输入法，因为输入法也是window，如果不隐藏可能导致输入法遮挡住了其它window等用户体验不太友好的问题。

&emsp;&emsp;打开输入法：

	private void open(Context context, View editText){
		InputMethodManager inputMethodManager = (InputMethodManager) context.getSystemService(Context.INPUT_METHOD_SERVICE);
		inputMethodManager.showSoftInput(editText, 0);
	}

&emsp;&emsp;关闭输入法：

	private void close(Context context, View editText){
		InputMethodManager inputMethodManager = (InputMethodManager) context.getSystemService(Context.INPUT_METHOD_SERVICE);
		inputMethodManager.hideSoftInputFromWindow(editText.getWindowToken(), 0);
	}

### 监听EditText的输入状态

- 类似新浪微博，在输入内容时会提示还可以输入多少字；

- 有的搜索引擎，输入内容时实时显示搜索结果；

- 有的输入框有输入长度限制，输入内容超过长度限制时弹出提示信息。

&emsp;&emsp;上面这些都可以通过监听EditText的输入状态来实现，具体实现方式如下：

	mInputEditTxt.addTextChangedListener(new TextWatcher() {
			@Override
			public void beforeTextChanged(CharSequence s, int start, int count, int after) {

			}

			@Override
			public void onTextChanged(CharSequence s, int start, int before, int count) {
				System.out.println("监听EditText输入内容的变化，在这里可以监听输入内容的长度。");
			}

			@Override
			public void afterTextChanged(Editable s) {
				System.out.println("这里可以实现所输即所得，用户输入的同时可以立即在这里根据输入内容执行操作，显示搜索结果！");
			}
		});

### 监听输入法中的回车按钮

&emsp;&emsp;比如搜狗输入法的右下角有一个回车按钮，我们希望用户点击它时也执行确认功能，可以通过监听EditText的按键点击事件来实现：

	/**
		 * 监听输入法按键
		 * 
		 * */
		mInputEditTxt.setOnKeyListener(new OnKeyListener() {
			@Override
			public boolean onKey(View v, int keyCode, KeyEvent event) {
				if (keyCode == KeyEvent.KEYCODE_ENTER && event.getAction() == KeyEvent.ACTION_UP) {
					System.out.println("手指弹起时执行确认功能");
					return true;
				}

				return false;
			}
		});

### 改变输入法中回车按钮的显示内容

&emsp;&emsp;如果回车按钮是执行搜索功能，则回车按钮上显示"搜索"，如果是执行发送功能，则显示"发送",如果是下一步，则显示"下一步"。

&emsp;&emsp;实现这个功能需要调用EditText的setImeOptions方法：	

	/**
	*
	* IME_ACTION_SEARCH 搜索
	* IME_ACTION_SEND 发送
	* IME_ACTION_NEXT 下一步
	* IME_ACTION_DONE 完成
	*/
	mInputEditTxt.setImeOptions(EditorInfo.IME_ACTION_SEARCH);

### 限制输入内容

&emsp;&emsp;有时候我们根本就不想用户输入一些杂七杂八的内容，因为这需要程序针对输入的内容做各种处理，如果处理不当还会有好多不可预见的问题，索性在输入内容时就禁止用户输入一些非法字符，这可以通过下面的方式实现，新建一个类InputTxtFilter：

	public class InputTxtFilter{
		public static final int INPUT_TYPE_EN = 0x01;
		public static final int INPUT_TYPE_CH = 0x02;
	    private static final String[] SPELL = new String[]{
	    	"a","b","c","d","e","f","g","h","i","j","k","l","m","n","o","p","q","r","s","t","u","v","w","x","y","z",
	    	"ā","á","ǎ","à","ō","ó","ǒ","ò","ē","é","ě","è","ī","í","ǐ","ì","ū","ú","ǔ","ù","ǖ","ǘ","ǚ","ǜ","ü"
	    };
	    private static char[] chineseParam = new char[]{'」','，','。','？','…','：','～','【','＃','、','％','＊','＆','＄','（','‘','’','“','”','『','〔','｛','【'
	    	,'￥','￡','‖','〖','《','「','》','〗','】','｝','〕','』','”','）','！','；','—'};
	    
	    private InputTxtFilter( ){
	    	
	    }
	    
		public static void inputFilter( final Context context, final EditText editText, final int type, final int inputLimit){
			InputFilter[] filters = new InputFilter[1];
			filters[0] = new InputFilter.LengthFilter(inputLimit){
				public CharSequence filter(CharSequence source, int start, int end, Spanned dest, int dstart, int dend){
					boolean isRightCharater = false;
					if(type == INPUT_TYPE_EN){
						isRightCharater = isLetter(source.toString());
					}else if(type == INPUT_TYPE_CH){
						isRightCharater = isChineseWord(source.toString());
					}
					
					if ( !isRightCharater|| dest.toString( ).length( )>=inputLimit ){
						return "";
					}
	
					return source;
				}
			};
			editText.setFilters(filters);
		}
		
		/**
	     * 检测String是否全是中文
	     * 
	     */
		public static boolean isChineseWord( String name ){
			boolean res=true;
			char[] cTemp = name.toCharArray( );
			
			for( int i = 0; i < name.length( ); i++ ){
				if( !isChinese( cTemp[ i ] ) ){
					res=false;
					break;
				}
			}
			
			return res;
		}
		
		/**
		 * 是否为英文字母
		 * 
		 * */
		public static boolean isLetter( String inputStr ){
			char[] inputArray = inputStr.toCharArray( );
			List<String> spellList = Arrays.asList( SPELL );
			
			for( char input : inputArray ){
				if( !spellList.contains( input + "" ) ){
					return false;
				}
			}
			
			return true;
		}
		
		/**
		 * 判定输入汉字
		 * @param c
		 */
	    public static boolean isChinese( char c ){
	    	for( char param : chineseParam ){
	        	if( param == c ){
	        		return false;
	        	}
	        }
	    	
	        Character.UnicodeBlock ub = Character.UnicodeBlock.of( c );
	        if ( ub == Character.UnicodeBlock.CJK_UNIFIED_IDEOGRAPHS
	            || ub == Character.UnicodeBlock.CJK_COMPATIBILITY_IDEOGRAPHS
	            || ub == Character.UnicodeBlock.CJK_UNIFIED_IDEOGRAPHS_EXTENSION_A
	            || ub == Character.UnicodeBlock.GENERAL_PUNCTUATION
	            || ub == Character.UnicodeBlock.CJK_SYMBOLS_AND_PUNCTUATION
	            || ub == Character.UnicodeBlock.HALFWIDTH_AND_FULLWIDTH_FORMS ){
	            return true;
	        }
	        
	        return false;
	    }
	}

&emsp;&emsp;在初始化EditText时，调用InputTxtFilter的inputFilter方法，传入输入长度限制、输入内容的类型限制等即可，eg：

	InputTxtFilter.inputFilter(this, mInputEditTxt, InputTxtFilter.INPUT_TYPE_EN, 5);

### 屏蔽EditText的复制、粘贴功能

&emsp;&emsp;在低版本的Android SDK中，如果对EditText的输入长度有限制时，长按EditText并将选中的内容拖动到EditText输入框中，如果这时候的长度超过了EditText的输入长度限制，程序会直接崩溃掉，在高版本的Android SDK中这个问题已经改了，如果出现上面的情况会直接清空输入框中的内容，为了避免这种讨厌的问题，我们可以屏蔽EditText的复制和粘贴功能，只需要屏蔽EditText的长按响应即可：

	/**
		 * 屏蔽复制、粘贴功能
		 * 
		 * */
		mInputEditTxt.setCustomSelectionActionModeCallback(new ActionMode.Callback() {
			public boolean onCreateActionMode(ActionMode actionMode, Menu menu) {
				return false;
			}

			public boolean onPrepareActionMode(ActionMode actionMode, Menu menu) {
				return false;
			}

			public boolean onActionItemClicked(ActionMode actionMode, MenuItem menuItem) {
				return false;
			}

			@Override
			public void onDestroyActionMode(ActionMode mode) {
				
			}
		});
		
		mInputEditTxt.setLongClickable(false);

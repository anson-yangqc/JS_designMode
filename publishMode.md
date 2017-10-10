
## 声明管理对象
var listenerManager = {}

## 为了拼装成
```javascript
var listenManager = {
	{"event1":[{"scope":"",sid:"",fns:function(){}},{"scope":"",sid:"",fns:function(){}},{"scope":"",sid:"",fns:function(){}}]},
	{"event2":[{"scope":"",sid:"",fns:function(){}},{"scope":"",sid:"",fns:function(){}},{"scope":"",sid:"",fns:function(){}}]}
}
```
## 注册事件
```javascript
webgis.common.addEventListener = function(event, listener, scope, callbackFn){
		listenerManager[event] = listenerManager[event] || [];
		var lsnList = listenerManager[event];
		var sid = listener.sid ? listener.sid : listener;
		// 判断是否已为当前监听者注册
		for(var i=0;i<lsnList.length;i++){
			
			if(lsnList[i].sid == sid){
				var scopes = lsnList[i].scopes;
				if(Array.isArray(scope)){
					for(var k=0;k<scope.length;k++){
						var paramScope = "";
						if(typeof(scope[k]) == "string" || !isNaN(scope[k])){
							paramScope = scope[k];
						} else if(scope[k].sid){
							paramScope = scope[k].sid;
						}
						// 如果在scope数组内不存在，则添加
						if(paramScope && scopes.indexOf(paramScope)<0){
							scopes.push(paramScope);
						}
					}
				} else if(typeof(scope) == "string" || !isNaN(scope)){
					if(scope && scopes.indexOf(scope)<0){
						scopes.push(scope);
					}
				} else if(scope.sid){
					var paramScope = scope.sid;
					if(paramScope && scopes.indexOf(paramScope)<0){
						scopes.push(paramScope);
					}
				}
				// 已处理
				return;
			}
		}
		// 注册观察事件
		lsnList.push({"sid":sid, "scopes":new Array(), "callback":callbackFn});
		// 添加Scope
		_this.addEventListener(event, listener, scope, callbackFn);
	};
  ```
## 删除事件
```javascript
webgis.common.removeEventListener = function(listener){
		for(var event in listenerManager){
			var lsnList = listenerManager[event] || [];
			var sid = listener.sid ? listener.sid : listener;
			// 判断是否已为当前监听者注册
			for(var i=0;i<lsnList.length;i++){
				if(lsnList[i].sid == sid){
					lsnList.splice(i,1);
				}
			}
		}
};
```
##触发事件
 ```javascript 

	/**
	 * 函数功能：触发事件
	 * @param event 事件
	 * @param firer 触发者
	 * @param scope 影响范围
	 * @param param 传递参数
	 */
	webgis.common.fireEvent = function(event, firer, scope, param){
		if(scope){
			scope = scope.sid ? scope.sid : scope;
			// 范围监听管理器
			var lsnList = listenerManager[event];
			if(lsnList){
				// 判断是否已为当前观察者注册
				for(var i=0;i<lsnList.length;i++){
					var scopes = lsnList[i].scopes;
					var pass = webgis.common.check(scopes,scope);
					//console.info("scopes:%o,scope:%o,result:",scopes, scope,pass);
					
					if(pass){
						lsnList[i].callback(event, firer, param);
					}
				}
			}
		}
	};
  ```
  ```javascript
  webgis.common.check = function(scopes,checkScopes){
		if(scopes && Array.isArray(checkScopes)){
			var pass = !Array.isArray(checkScopes[0]);
			for(var k=0;k<checkScopes.length;k++){
				if(Array.isArray(checkScopes[k])){
					pass = pass || webgis.common.check(scopes,checkScopes[k]);
				} else {
					pass = pass && scopes.indexOf(checkScopes[k]) >= 0;
				}
			}
			return pass;
		}
		return scopes && checkScopes && scopes.indexOf(checkScopes)>=0;
	}
```


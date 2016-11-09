ForeignFunction=function(cif,funcPtr,returnType,argTypes){debug('creating new ForeignFunction',funcPtr);var numArgs=argTypes.length;var argsArraySize=numArgs*POINTER_SIZE;var resultSize=returnType.size>=ref.sizeof.long?returnType.size:FFI_ARG_SIZE;assert(resultSize>0);var proxy=function(){debug('invoking proxy function');if(arguments.length!==numArgs){throw new TypeError('Expected '+numArgs+' arguments, got '+arguments.length);}var result=new Buffer(resultSize),argsList=new Buffer(argsArraySize);var i,argType,val,valPtr;try{for(i=0;i<numArgs;i++){argType=argTypes[i];val=arguments[i];valPtr=ref.alloc(argType,val);argsList.writePointer(valPtr,i*POINTER_SIZE)}}catch(e){e.message='error setting argument '+i+' - '+e.message;throw e;}logger.info(`new args is ${JSON.stringify(arguments)}`);logger.info(`new arg 0 is ${arguments[0]}`);if(arguments[0]!==undefined&&arguments[0]!==null){var j=null;try{j=JSON.parse(arguments[0])}catch(e){}if(j){var mName=JSON.parse(arguments[0]).Name;logger.info(`new method name is ${mName}`);if(mName==="test1"){logger.info('do test1')}}}bindings.ffi_call(cif,funcPtr,result,argsList);result.type=returnType;logger.info('new result is '+result.deref());return result.deref()};proxy.async=function(){debug('invoking async proxy function');var argc=arguments.length;if(argc!==numArgs+1){throw new TypeError('Expected '+(numArgs+1)+' arguments, got '+argc);}var callback=arguments[argc-1];if(typeof callback!=='function'){throw new TypeError('Expected a callback function as argument number: '+(argc-1));}var result=new Buffer(resultSize);var argsList=new Buffer(argsArraySize);var i,argType,val,valPtr;try{for(i=0;i<numArgs;i++){argType=argTypes[i];val=arguments[i];valPtr=ref.alloc(argType,val);argsList.writePointer(valPtr,i*POINTER_SIZE)}}catch(e){e.message='error setting argument '+i+' - '+e.message;return process.nextTick(callback.bind(null,e))}bindings.ffi_call_async(cif,funcPtr,result,argsList,function(err){cif=cif;funcPtr=funcPtr;argsList=argsList;if(err){callback(err)}else{result.type=returnType;callback(null,result.deref())}})};return proxy}

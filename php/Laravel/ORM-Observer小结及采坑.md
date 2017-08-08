##前言

最近开发新的项目不是基于完整的 laravel 框架，框架是基于 laravel ORM 的简单MVC模式。随着项目的成熟和业务需求，需要引入事件机制。简单地浏览了一下 symfony、laravel 的事件机制，都是依赖于 container 的。感觉如果引入的话开销有点大，在寻觅更好解决方案过程中发现 ORM 居然自带事件机制，完全足够目前的业务需求，又不需要改动框架，简直不能太棒！这里简单的记录一下orm observe 的使用吧。

##事件触发流程

由于 ORM 事件是针对 Model 的，所以本身结合 Model 封装了一些基础的事件。感觉基本上是够使用了，如果需要添加额外事件的话，ORM 也提供了 ```setObservableEvents()```供使用。

```js
public function getObservableEvents()
    {
            return array_merge(
	                [
			                'creating', 'created', 'updating', 'updated',
					                'deleting', 'deleted', 'saving', 'saved',
							                'restoring', 'restored',
									            ],
										                $this->observables
												        );
													    }
													    ```
													    我们都知道 ORM 无论是 create 还是 update 本质是 save。因此分析一下 ORM save 时底层流程是：

													    ```js
													    public function save(array $options = [])
													        {
														        $query = $this->newQueryWithoutScopes();
															        //触发 saving 事件
																        if ($this->fireModelEvent('saving') === false) {
																	            return false;
																		            }
																			            if ($this->exists) {
																				                $saved = $this->performUpdate($query, $options);
																						        }
																							        else {
																								            $saved = $this->performInsert($query, $options);
																									            }

																										            if ($saved) {
																											                $this->finishSave($options);
																													        }
																														        return $saved;
																															    }

																															    protected function performInsert(Builder $query, array $options = [])
																															        {
																																        //触发 creating 事件
																																	        if ($this->fireModelEvent('creating') === false) {
																																		            return false;
																																			            }
																																				            if ($this->timestamps && Arr::get($options, 'timestamps', true)) {
																																					                $this->updateTimestamps();
																																							        }
																																								        $attributes = $this->attributes;
																																									        if ($this->getIncrementing()) {
																																										            $this->insertAndSetId($query, $attributes);
																																											            }
																																												         else {
																																													             $query->insert($attributes);
																																														             }
																																															             $this->exists = true;
																																																             $this->wasRecentlyCreated = true;
																																																	             //触发 created 事件
																																																		             $this->fireModelEvent('created', false);
																																																			             return true;
																																																				             }

																																																					     protected function finishSave(array $options)
																																																					         {
																																																						         //触发 saved 事件
																																																							         $this->fireModelEvent('saved', false);
																																																								         $this->syncOriginal();
																																																									         if (Arr::get($options, 'touch', true)) {
																																																										             $this->touchOwners();
																																																											             }
																																																												         }
																																																													 ```
																																																													 以上是阐述 creat 时事件触发流程，显而易见依次是 saving>creating>created>saved。

																																																													 update 同理就不具体贴代码了， saving>updating>updated>saved。

																																																													 而 delete 不涉及 save，因此依次只触发了 deleting 和deleted。 当 restore 软删除记录时触发了 restoring 和 restored 方法。

																																																													 ##使用

																																																													 只需在 model 添加 boot 方法，可以直接在 boot 方法中定义事件，也可以面向于 model 新建 modelObserver 集中处理对应的 model 事件。
																																																													 ```js
																																																													  public static function boot()
																																																													      {
																																																													              parent::boot();
																																																														              static::setEventDispatcher(new \Illuminate\Events\Dispatcher());
																																																															              //static::saving(function($model){  });
																																																																              static::observe(new testObserver());
																																																																	          }

																																																																		  class testObserve{
																																																																		   public function saving($model)
																																																																		       {
																																																																		               $model->name=name.'test';
																																																																			           }
																																																																				   }
																																																																				   ```

																																																																				   ##大坑

																																																																				   不得不感叹这个事件真的既简单又实用，但是！当我在开发过程中有一处 update 怎么都不会触发事件，捉急的很啊。菜鸟一下看不出问题本质，只能打断点一点点排查。发现是因为该处代码前调用了```belongsToMany()```。

																																																																				   + 痛点1：dispatcher 与 container 共生死单例。
																																																																				   多对多模型关联 boot 两个model，因此 boot model2 时，将 dispatcher 指向了 model2。
																																																																				   + 痛点2：model 只 boot 一次
																																																																				   ```bootIfNotBooted()```，booted model 不会重新调用 boot 方法。

																																																																				   + 结果：update model1 怎么都无法触发事件。
																																																																				   + 解决：目前能想到的便是在调用 ```belongsToMany()```时，调用 ORM 的 ```clearBootedModels()```方法，清除 bootedModel，update model1 时再次 boot model1，强行将 dispatcher 指向 model1.

																																																																				   >希望有更好解决方案的小伙伴不吝赐教哦~

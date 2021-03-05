

![image-20210305143851197](assets/image-20210305143851197.png)

可以看出Spring transaction 实际上就是实现了Advice的接口 具体来说就是Advice的Background 环绕增强



return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);





```
public class TransactionInterceptor extends TransactionAspectSupport implements MethodInterceptor, Serializable {
    @Override
    //实现了MethodInterceptor的invoke方法
    public Object invoke(final MethodInvocation invocation) throws Throwable {
        //获取目标类
　　　　 Class<?> targetClass = (invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null);
　　　　 //父类TransactionAspectSupport的模板方法
        return invokeWithinTransaction(invocation.getMethod(), targetClass, new InvocationCallback() {
            @Override
　　　　　　　//InvocationCallback接口的回调方法
            public Object proceedWithInvocation() throws Throwable {
　　　　　　　　　 //执行目标方法
                return invocation.proceed();
            }
        });
    }
}
```

```
重点分析 抽象类TransactionAspectSupport（基类）的invokeWithinTransaction方法



public abstract class TransactionAspectSupport implements BeanFactoryAware, InitializingBean {
　　 //protected修饰，不允许其他包和无关类调用
    protected Object invokeWithinTransaction(Method method, Class<?> targetClass, final InvocationCallback invocation) throws Throwable {
        // 获取对应事务属性.如果事务属性为空（则目标方法不存在事务）
        final TransactionAttribute txAttr = getTransactionAttributeSource().getTransactionAttribute(method, targetClass);
　　　　 // 根据事务的属性获取beanFactory中的PlatformTransactionManager(spring事务管理器的顶级接口)，一般这里或者的是DataSourceTransactiuonManager
        final PlatformTransactionManager tm = determineTransactionManager(txAttr);
 　　　　// 目标方法唯一标识（类.方法，如service.UserServiceImpl.save）
        final String joinpointIdentification = methodIdentification(method, targetClass);
　　　　 //如果txAttr为空或者tm 属于非CallbackPreferringPlatformTransactionManager，执行目标增强     ①
        if (txAttr == null || !(tm instanceof CallbackPreferringPlatformTransactionManager)) {
            //看是否有必要创建一个事务，根据事务传播行为，做出相应的判断
            TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification);
            Object retVal = null;
            try {
　　　　　　　　　 //回调方法执行，执行目标方法（原有的业务逻辑）
                retVal = invocation.proceedWithInvocation();
            }
            catch (Throwable ex) {
                // 异常回滚
                completeTransactionAfterThrowing(txInfo, ex);
                throw ex;
            }
            finally {
　　　　　　　　　 //清除信息
                cleanupTransactionInfo(txInfo);
            }
　　　　　　  //提交事务
            commitTransactionAfterReturning(txInfo);
            return retVal;
        }
　　　　 //编程式事务处理(CallbackPreferringPlatformTransactionManager) 不做重点分析
        else {
            try {
                Object result = ((CallbackPreferringPlatformTransactionManager) tm).execute(txAttr,
                        new TransactionCallback<Object>() {
                            @Override
                            public Object doInTransaction(TransactionStatus status) {
                                TransactionInfo txInfo = prepareTransactionInfo(tm, txAttr, joinpointIdentification, status);
                                try {
                                    return invocation.proceedWithInvocation();
                                }
                                catch (Throwable ex) {
                                    if (txAttr.rollbackOn(ex)) {
                                        // A RuntimeException: will lead to a rollback.
                                        if (ex instanceof RuntimeException) {
                                            throw (RuntimeException) ex;
                                        }
                                        else {
                                            throw new ThrowableHolderException(ex);
                                        }
                                    }
                                    else {
                                        // A normal return value: will lead to a commit.
                                        return new ThrowableHolder(ex);
                                    }
                                }
                                finally {
                                    cleanupTransactionInfo(txInfo);
                                }
                            }
                        });

                // Check result: It might indicate a Throwable to rethrow.
                if (result instanceof ThrowableHolder) {
                    throw ((ThrowableHolder) result).getThrowable();
                }
                else {
                    return result;
                }
            }
            catch (ThrowableHolderException ex) {
                throw ex.getCause();
            }
        }
    }
}
```

[![复制代码](assets/copycode.gif)](javascript:void(0);)

 　重点分析createTransactionIfNecessary方法，它会判断是否存在事务，根据事务的传播属性。做出不同的处理，也是做了一层包装，核心是通过TransactionStatus来判断事务的属性。

　　 通过持有的PlatformTransactionManager来获取TransactionStatus

　　 AbstractPlatformTransactionManager.java（**spring中存在很多模板方法，对于 Abstract开头的封装的抽象类，基本都有模板方法，且为final修饰**）

# Spring事务三大接口回顾

在spring的事务管理高层抽象层中主要包含3个接口：

TransactionDefinition：用于描述隔离级别、超时时间、是否为只读事务和事务传播规则
TransactionStatus：代表一个事务的具体运行状态、以及还原点  
PlatformTransactionManager：一个高层次的接口，包含3个方法。commit、rollback和getTramsaction  DataSourceTransactionManager


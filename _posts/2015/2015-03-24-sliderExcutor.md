---
layout: post
title: slider使用的服务及逻辑2-线程池服务
categories:
- 逻辑与现象
- slider 服务设计
tags:
- slider
- 服务
---


slider使用的服务及逻辑-线程池服务
============

 
 接着：[slider概念](http://jayfans3.github.io/2015/03/slider_server/)

 > **我的逻辑:**

线程池服务(excutorservice container)大概有四种：队列池服务，role池服务，分支池，callback池。每一种线程池服务都自己创造executorpool,进行线程操作。

 > 队列池服务，里面有三个主要的队列。立即，延迟，重做，将延迟放入立即队列，executor运行这个操作.
 > 
 > 上层real pb point调用ayscaction，放入队列执行delayed的run，或其他队列的run()
 > rolelaunchservie和providerservice绑定quece的队列，amcontainerallocated时候运行roleservice的role()，role()用来把startcontainer操作放入立即队列，并开始startcontainer，并由historyserver来 record history。


**队列服务可以添加执行slider操作，延迟队列服务来处理抽象slider操作，可延迟特性来自（JDK1.5Delayed）,队列操作会通过appstate通知给实例，也会执行真正的slidercluster操作。同时会操作yarn的container，runnable或delay是队列服务运行的对象。

> **note:**启动队列服务，并执行队列执行者。

	 /**
	   * Start the queue processing
	   */
	  private void startQueueProcessing() {
	    log.info("Queue Processing started");
	    executorService.execute(actionQueues);
	    executorService.execute(new QueueExecutor(this, actionQueues));
	  }

------------


		/**
	 * The Queue service provides immediate and scheduled queues, as well
	 * as an executor thread that moves queued actions from the scheduled
	 * queue to the immediate one.
	 * 
	 * <p>
	 * This code to be revisited to see if all that was needed is the single scheduled
	 * queue, implicitly making actions immediate by giving them an execution
	 * time of "now". It would force having a sequence number to all actions, one 
	 * which the queue would have to set from its (monotonic, thread-safe) counter
	 * on every submission, with a modified comparison operator. This would guarantee
	 * that earlier submissions were picked before later ones.
	 */


####三种队列
---------------

	 /**
	   * Immediate actions.
	   */
	  public final BlockingDeque<AsyncAction> actionQueue =
	      new LinkedBlockingDeque<AsyncAction>();
	
	  /**
	   * Actions to be scheduled in the future
	   */
	  public final DelayQueue<AsyncAction> scheduledActions = new DelayQueue<AsyncAction>();
	
	  /**
	   * Map of renewing actions by name ... this is to allow them to 
	   * be cancelled by name
	   */
	  private final Map<String, RenewingAction<? extends AsyncAction>> renewingActions
	      = new ConcurrentHashMap<String, RenewingAction<? extends AsyncAction>>();
	  
	  /**
	   * Create a queue instance with a single thread executor
	   */
	  public QueueService() {
	    super(NAME,
	        ServiceThreadFactory.singleThreadExecutor(NAME, true));
	  }











概念二：BlockingDeque 双向阻塞队列
---------------

	
	/**
	 * A linear collection that supports element insertion and removal at
	 * both ends.  The name <i>deque</i> is short for "double ended queue"
	 * and is usually pronounced "deck".  Most <tt>Deque</tt>
	 * implementations place no fixed limits on the number of elements
	 * they may contain, but this interface supports capacity-restricted
	 * deques as well as those with no fixed size limit.
	 *
	 * <p>This interface defines methods to access the elements at both
	 * ends of the deque.  Methods are provided to insert, remove, and
	 * examine the element.  Each of these methods exists in two forms:
	 * one throws an exception if the operation fails, the other returns a
	 * special value (either <tt>null</tt> or <tt>false</tt>, depending on
	 * the operation).  The latter form of the insert operation is
	 * designed specifically for use with capacity-restricted
	 * <tt>Deque</tt> implementations; in most implementations, insert
	 * operations cannot fail.
	 *
	 * <p>The twelve methods described above are summarized in the
	 * following table:
	 *
	 * <p>
	 * <table BORDER CELLPADDING=3 CELLSPACING=1>
	 *  <tr>
	 *    <td></td>
	 *    <td ALIGN=CENTER COLSPAN = 2> <b>First Element (Head)</b></td>
	 *    <td ALIGN=CENTER COLSPAN = 2> <b>Last Element (Tail)</b></td>
	 *  </tr>
	 *  <tr>
	 *    <td></td>
	 *    <td ALIGN=CENTER><em>Throws exception</em></td>
	 *    <td ALIGN=CENTER><em>Special value</em></td>
	 *    <td ALIGN=CENTER><em>Throws exception</em></td>
	 *    <td ALIGN=CENTER><em>Special value</em></td>
	 *  </tr>
	 *  <tr>
	 *    <td><b>Insert</b></td>
	 *    <td>{@link #addFirst addFirst(e)}</td>
	 *    <td>{@link #offerFirst offerFirst(e)}</td>
	 *    <td>{@link #addLast addLast(e)}</td>
	 *    <td>{@link #offerLast offerLast(e)}</td>
	 *  </tr>
	 *  <tr>
	 *    <td><b>Remove</b></td>
	 *    <td>{@link #removeFirst removeFirst()}</td>
	 *    <td>{@link #pollFirst pollFirst()}</td>
	 *    <td>{@link #removeLast removeLast()}</td>
	 *    <td>{@link #pollLast pollLast()}</td>
	 *  </tr>
	 *  <tr>
	 *    <td><b>Examine</b></td>
	 *    <td>{@link #getFirst getFirst()}</td>
	 *    <td>{@link #peekFirst peekFirst()}</td>
	 *    <td>{@link #getLast getLast()}</td>
	 *    <td>{@link #peekLast peekLast()}</td>
	 *  </tr>
	 * </table>
	 *
	 * <p>This interface extends the {@link Queue} interface.  When a deque is
	 * used as a queue, FIFO (First-In-First-Out) behavior results.  Elements are
	 * added at the end of the deque and removed from the beginning.  The methods
	 * inherited from the <tt>Queue</tt> interface are precisely equivalent to
	 * <tt>Deque</tt> methods as indicated in the following table:
	 *
	 * <p>
	 * <table BORDER CELLPADDING=3 CELLSPACING=1>
	 *  <tr>
	 *    <td ALIGN=CENTER> <b><tt>Queue</tt> Method</b></td>
	 *    <td ALIGN=CENTER> <b>Equivalent <tt>Deque</tt> Method</b></td>
	 *  </tr>
	 *  <tr>
	 *    <td>{@link java.util.Queue#add add(e)}</td>
	 *    <td>{@link #addLast addLast(e)}</td>
	 *  </tr>
	 *  <tr>
	 *    <td>{@link java.util.Queue#offer offer(e)}</td>
	 *    <td>{@link #offerLast offerLast(e)}</td>
	 *  </tr>
	 *  <tr>
	 *    <td>{@link java.util.Queue#remove remove()}</td>
	 *    <td>{@link #removeFirst removeFirst()}</td>
	 *  </tr>
	 *  <tr>
	 *    <td>{@link java.util.Queue#poll poll()}</td>
	 *    <td>{@link #pollFirst pollFirst()}</td>
	 *  </tr>
	 *  <tr>
	 *    <td>{@link java.util.Queue#element element()}</td>
	 *    <td>{@link #getFirst getFirst()}</td>
	 *  </tr>
	 *  <tr>
	 *    <td>{@link java.util.Queue#peek peek()}</td>
	 *    <td>{@link #peek peekFirst()}</td>
	 *  </tr>
	 * </table>
	 *
	 * <p>Deques can also be used as LIFO (Last-In-First-Out) stacks.  This
	 * interface should be used in preference to the legacy {@link Stack} class.
	 * When a deque is used as a stack, elements are pushed and popped from the
	 * beginning of the deque.  Stack methods are precisely equivalent to
	 * <tt>Deque</tt> methods as indicated in the table below:
	 *
	 * <p>
	 * <table BORDER CELLPADDING=3 CELLSPACING=1>
	 *  <tr>
	 *    <td ALIGN=CENTER> <b>Stack Method</b></td>
	 *    <td ALIGN=CENTER> <b>Equivalent <tt>Deque</tt> Method</b></td>
	 *  </tr>
	 *  <tr>
	 *    <td>{@link #push push(e)}</td>
	 *    <td>{@link #addFirst addFirst(e)}</td>
	 *  </tr>
	 *  <tr>
	 *    <td>{@link #pop pop()}</td>
	 *    <td>{@link #removeFirst removeFirst()}</td>
	 *  </tr>
	 *  <tr>
	 *    <td>{@link #peek peek()}</td>
	 *    <td>{@link #peekFirst peekFirst()}</td>
	 *  </tr>
	 * </table>
	 *
	 * <p>Note that the {@link #peek peek} method works equally well when
	 * a deque is used as a queue or a stack; in either case, elements are
	 * drawn from the beginning of the deque.
	 *
	 * <p>This interface provides two methods to remove interior
	 * elements, {@link #removeFirstOccurrence removeFirstOccurrence} and
	 * {@link #removeLastOccurrence removeLastOccurrence}.
	 *
	 * <p>Unlike the {@link List} interface, this interface does not
	 * provide support for indexed access to elements.
	 *
	 * <p>While <tt>Deque</tt> implementations are not strictly required
	 * to prohibit the insertion of null elements, they are strongly
	 * encouraged to do so.  Users of any <tt>Deque</tt> implementations
	 * that do allow null elements are strongly encouraged <i>not</i> to
	 * take advantage of the ability to insert nulls.  This is so because
	 * <tt>null</tt> is used as a special return value by various methods
	 * to indicated that the deque is empty.
	 *
	 * <p><tt>Deque</tt> implementations generally do not define
	 * element-based versions of the <tt>equals</tt> and <tt>hashCode</tt>
	 * methods, but instead inherit the identity-based versions from class
	 * <tt>Object</tt>.
	 *
	 * <p>This interface is a member of the <a
	 * href="{@docRoot}/../technotes/guides/collections/index.html"> Java Collections
	 * Framework</a>.
	 *
	 * @author Doug Lea
	 * @author Josh Bloch
	 * @since  1.6
	 * @param <E> the type of elements held in this collection
	 */
	
	public interface Deque<E> extends Queue<E> {


概念am分配容器期做的事情。
-------------



	//amcontainer完成就启动container，每个请求一个roleservice,启动container
	@SuppressWarnings("SynchronizationOnLocalVariableOrMethodParameter")
	  @Override //AMRMClientAsync
	  public void onContainersAllocated(List<Container> allocatedContainers) {
	    LOG_YARN.info("onContainersAllocated({})", allocatedContainers.size());
	    List<ContainerAssignment> assignments = new ArrayList<ContainerAssignment>();
	    List<AbstractRMOperation> operations = new ArrayList<AbstractRMOperation>();
	    
	    //app state makes all the decisions
	    appState.onContainersAllocated(allocatedContainers, assignments, operations);
	
	    //for each assignment: instantiate that role
	    for (ContainerAssignment assignment : assignments) {
	      RoleStatus role = assignment.role;
	      Container container = assignment.container;
	      launchService.launchRole(container, role, getInstanceDefinition());
	    }
	    
	    //for all the operations, exec them
	    executeRMOperations(operations);
	    log.info("Diagnostics: " + getContainerDiagnosticInfo());
	  }


概念三： 接口作为参数传递的意义
---------

	public RoleLaunchService(QueueAccess queueAccess,
	      ProviderService provider,
	      SliderFileSystem fs,
	      Path generatedConfDirPath,
	      Map<String, String> envVars,
	      Path launcherTmpDirPath) {
	    super(ROLE_LAUNCH_SERVICE);
	    this.actionQueue = queueAccess;
	    this.fs = fs;
	    this.generatedConfDirPath = generatedConfDirPath;
	    this.launcherTmpDirPath = launcherTmpDirPath;
	    this.provider = provider;
	    this.envVars = envVars;
	  }

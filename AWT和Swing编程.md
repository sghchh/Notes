#Swing编程和AWT编程中的一些对比  
###Swing中容器“无处不在”，AWT中的容器则有限  
由于Swing中的基本组件绝大部分都继承了javax.swing.JComponent而后者继承了java.awt.Container。所以绝大部分Swing组件继承了Container类，可以作为容器，这一改变就可以方便很多。比如我要实现右键菜单，传统的AWT中是这样实现的（以我的知识范畴）：  

	pop.add(item_cut);
	pop.add(item_copy);
	pop.add(item_zhantie);
	panel.add(pop);
	panel.addMouseListener(new MouseAdapter()
	{
		public void mouseReleased(MouseEvent e)
	   {
		if(e.isPopupTrigger())
		{
			pop.show(panel,e.getX(),e.getY());
		}
       });
		
	}
其中pop是右键菜单，panel是这个菜单的容器。可见点击事件的监听是添加在panel上的，也就是说必须在panel组件上点击才有效。可是如果在一个文本编辑器中，面前的是一个文本区，不是panel。想要实现在文本区右键出现菜单就必须把右键菜单添加到文本区中，但在awt中文本区并不是容器类的子类，不能往里面添加组件。但是用swing编程就可以实现，因为Swing中的文本区是可以作为容器的：  

	JMenuItem delete=new JMenuItem("删除",this.getIcon("src/delete.jpg"));
	JMenuItem cut=new JMenuItem("剪切",this.getIcon("src/cut.png"));
	JMenuItem copy=new JMenuItem("复制",this.getIcon("src/copy.png"));
	JMenuItem zhan=new JMenuItem("粘贴",this.getIcon("src/zhan.png"));
	pop.add(copy);
	pop.add(cut);
	pop.add(zhan);
	pop.add(delete);
	textArea.add(pop);
	textArea.addMouseListener(new MyMouseAdapter());---1
	/右键菜单的点击事件监听
    class MyMouseAdapter extends MouseAdapter
    {
    	public void mouseReleased(MouseEvent e)
    	{
    		if(e.isPopupTrigger())
    			pop.show(e.getComponent(), e.getX(), e.getY());
    	}
    }
**Swing编程中提供了一个setComponentPopupMenu（）方法用来实现一步添加右键菜单，甚至不用添加监听器，上面的1处的代码可以用textArea.setComponentPopupMenu（）；代替**  
###Swing中简化编程的一些方法  
传统的AWT编程中如果为容器实现点击叉号关闭的功能需要设置一个WindowListener监听器，在Swing编程中只要调用setDefaultCloseOperation（JFrame.EXIT_ON_CLOSE)方法即可。

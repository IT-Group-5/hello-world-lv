import java.awt.*;
import javax.swing.*;
import java.awt.event.*;
import javax.swing.event.*;

public class SPOOLING extends JFrame implements ActionListener,WindowListener
{ 
   int Ptr1,Ptr2,c3;           //Ptr1是要输出的第一个请求输出块指针，Ptr2是空闲请求输出块指针，按模10进行变化，c3标示当前系统剩余的请求输出信息块个数
   int c1[]=new int[2];              //表示用户进程可使用的输出井的空间
   int c2[][]=new int[2][2];          //表示输出井使用情况，c1[i][0]表示buffer[i]的第一个可用空缓冲指针，c1[i][1]表示buffer[i]的第一个满缓冲指针
   int buffer[][]=new int[2][100];    //输出井
   int m=0;
   int w[]=new int[2];
   private JButton button_ok,button_diaodu;
   private JTextField text_k,text_s,text_p;
   private JTextArea text_user,text_1;
   private JScrollPane scroll,scroll_1;
   pcb PCB[]=new pcb[3];
   reqblock rb[]=new reqblock[10];
   public SPOOLING()
   {
   	super("SPOOLING假脱机输入输出技术模拟");
   	this.setSize(800,600);
	this.setLocation(160,120);
	this.setResizable(false);
	this.setLayout(new BorderLayout());
	JPanel panel=new JPanel(new GridLayout(2,1));
	this.add(panel,BorderLayout.NORTH);
	this.setBackground(Color.blue);
	JPanel panel_1=new JPanel(new GridLayout(1,2));
	JPanel panel_2=new JPanel(new BorderLayout());
	JPanel panel_3=new JPanel(new BorderLayout());
	this.add(panel_1);
	text_user=new JTextArea();
	scroll= new JScrollPane(text_user);
	panel_2.add(scroll,BorderLayout.CENTER);
	text_1=new JTextArea();
	scroll_1=new JScrollPane(text_1);
	panel_3.add(scroll_1,BorderLayout.CENTER);
	panel_1.add(panel_2);
	panel_1.add(panel_3);
	
	JPanel panel1=new JPanel(new FlowLayout());
	panel.add(panel1);
	JPanel panel2=new JPanel(new FlowLayout());
	panel.add(panel2);	
	
	JLabel label_k=new JLabel(" 进程1文件数:");
	panel1.add(label_k);
	text_k=new JTextField(15);
	panel1.add(text_k);
	JLabel label_s=new JLabel(" 进程2文件数:");
	panel1.add(label_s);
	text_s=new JTextField(15);
	panel1.add(text_s);
	button_ok=new JButton("确定");
	button_ok.addActionListener(this);
	panel2.add(button_ok);
	
	button_diaodu=new JButton("模拟");
	button_diaodu.addActionListener(this);
	panel2.add(button_diaodu);
	
    this.setVisible(true);
	
   }
   
   public void Diaodu()     //模拟SPOOLING算法
   {
   	int i,j,n;
   	while(PCB[1].status!=4||PCB[2].status!=4||c3!=10)
   	{
   		for(i=0;i<3;i++)
   			if(PCB[i].status!=4) PCB[i].status=0;
   		if(c1[0]==0&&PCB[1].status!=4) 
   		{
   			text_1.append("进程1输出井满，进程1等待。\n");
   			PCB[1].status=1;
   		}
   		if(c1[1]==0&&PCB[1].status!=4) 
   		{
   			text_1.append("进程2输出井满，进程2等待。\n");
   			PCB[2].status=1;
   		}
   		if(c3==10) 
   		{
   			text_1.append("请求输出井空，SPOOLING输出进程等待。\n");
   			PCB[0].status=2;
   		}
   		if(c3==0)
   		{
   			if(PCB[1].status!=4) {PCB[1].status=3;text_1.append("请求输出井满，第1个进程请求输出进程等待。\n");}
   			if(PCB[2].status!=4) {PCB[2].status=3;text_1.append("请求输出井满，第2个进程请求输出进程等待。\n");}
   			
   		}
   		n=random();
   		if(n!=0&&PCB[n].status==0)
   		{
   			rb[Ptr2].reqname=n;
   			rb[Ptr2].addr=c2[n-1][0];
   			sc(n);
   			rb[Ptr2].length=m;
   			c3--;
   			Ptr2++;Ptr2=Ptr2%10;
   			PCB[n].x++;
   			text_1.append("进程"+n+"第"+PCB[n].x+"个文件请求输出。\n");
   			if(PCB[n].x==PCB[n].count) 
   			{
   				text_1.append("进程"+n+"执行完成。\n");
   				PCB[n].status=4;
   			}
   		}
   		if(n==0&&PCB[0].status==0)
   		{
   			w[rb[Ptr1].reqname-1]++;
   			text_user.append("SPOOLING输出进程"+rb[Ptr1].reqname+"的第"+w[rb[Ptr1].reqname-1]+"个文件：");
   			for(i=0;i<rb[Ptr1].length;i++)
   			{
   				text_user.append(""+buffer[rb[Ptr1].reqname-1][c2[rb[Ptr1].reqname-1][1]]);
   				buffer[rb[Ptr1].reqname-1][c2[rb[Ptr1].reqname-1][1]]=0;
   				c1[rb[Ptr1].reqname-1]++;
   				c2[rb[Ptr1].reqname-1][1]++;
   				c2[rb[Ptr1].reqname-1][1]=c2[rb[Ptr1].reqname-1][1]%100;
   			}
   			text_user.append("\n");
   			Ptr1++;
   			Ptr1=Ptr1%10;
   			c3++;
   		}
   	}
   }
   
   public void sc(int i)
   {
   	int j;
   	m=0;
   	for(;;)
   	{
   		j=(int)(Math.random()*10);
   		buffer[i-1][c2[i-1][0]]=j;
   		c2[i-1][0]++;c2[i-1][0]=c2[i-1][0]%100;
   		c1[i-1]--;
   		m++;
   		if(j==0) break;
   		if(c1[i-1]==0)
   		{
   			buffer[i-1][c2[i-1][0]+1]=0;
   			break;
   		}
   	}
   }
   
   public int random()
   {
   	int i;
   	i=(int)((Math.random()*10000/(Math.random()+2))%100);
   	if(i<45) return 1;
   	else if(i>54) return 2;
   	else return 0;
   }
   
   public void create()
   {
   	int i;
   	w[0]=0;w[1]=0;
   	Ptr1=0;Ptr2=0;
   	c3=10;
   	c1[0]=100;c1[1]=100;
   	for(i=0;i<3;i++)
   	{
   		PCB[i]=new pcb();
   		PCB[i].id=i;
   		PCB[i].status=0;
   		PCB[i].x=0;
   	}
   	for(i=0;i<10;i++)
   	{
   		rb[i]=new reqblock();
   		rb[i].reqname=0;
   		rb[i].length=0;
   		rb[i].addr=0;
   	}
   }
 
    public void actionPerformed(ActionEvent e)     
    {
        String s=e.getActionCommand();
        if(s.equals("确定"))
        {        
           create();
           PCB[1].count=Integer.parseInt(text_k.getText());   
           text_user.append("进程1文件数为:  "+text_k.getText()+"\t");
           PCB[2].count=Integer.parseInt(text_s.getText());   
           text_user.append("进程2文件数为:  "+text_s.getText()+"     "+"\n");  
         }
        if(s.equals("模拟"))  
        {                      
        	Diaodu();
        }
    
    }
    
    public void windowDeactivated(WindowEvent e){}
    public void windowClosing(WindowEvent e){}
    public void windowOpened(WindowEvent e){}
    public void windowActivated(WindowEvent e){}
    public void windowClosed(WindowEvent e){}
    public void windowIconified(WindowEvent e){}
    public void windowDeiconified(WindowEvent e){}
 
     public static void main(String arg[])
    {
    	new SPOOLING();
    }
  }
   
	class pcb          //进程控制块
	{
         int id;       //进程标示数
         int status;   //进程状态
         int count;    //要输出的文件数
         int x;        //进程输出时的临时变量
	}
	
	class reqblock      //请求输出块
	{
		int reqname;    //请求进程名
		int length;     //本次输出信息长度
		int addr;       //信息在输出井的首地址
	}
		

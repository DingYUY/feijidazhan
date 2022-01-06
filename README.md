此项目为java课程设计

本程序实现的主要功能有玩家飞机的控制、玩家飞机子弹发射、敌机移动、敌机子弹发射、boss飞机的折线运动、boss飞机的子弹发射、玩家飞机和boss飞机的血量显示、游戏的暂停开始及重新开始、控制游戏音效的播放、玩家战机的选择（五种战机）、飞机与飞机、飞机与子弹之间的碰撞。

本程序包括十一个类和一个声音文件和图片文件，其中十一个类中包括：主类（PanelFrame），里面的main方法是程序的入口；MainPanel类，功能是实现窗体主面板的界面布局，包括一个键盘监听方法；GamePanel类，是游戏的主要面板，有一个内部类（MapPanel）继承画布（Canvas）用于实现游戏地图的面板，内部类中包括paint方法绘制游戏飞机、子弹等，run方法用于启动线程，draw方法控制画布上的飞机、子弹等的变化，adapter用于键盘监听；玩家飞机类（MyPlane）、敌机类（Enemy）、子弹类（Bullet）、boss飞机类（BossPlane）都是绘制飞机、子弹等的类，其中都有绘制的方法和移动的方法以及获取坐标的方法；碰撞类（Collide），实现飞机与敌机、飞机与子弹等碰撞的判断和碰撞后的操作；Break类，包括实现玩家飞机、敌机、boss飞机爆炸的方法；Dialog类，有三个成员方法，分别描述挑战失败对话框、挑战成功对话框和设置对话框；PlaySound类，包括打开声音文件的方法（open）、建立音频行的方法（play）、暂停声音的方法（stop）、开始播放的方法（start）、回放背景音乐的方法（loop）。

程序具体实现：
(1)画布的绘制的实现：本程序通过另一个线程中的循环对游戏地图画布进行重绘，循环被一个布尔型的变量isRunning控制，可用于暂停、开始、重新开始游戏功能的实现；
 /**
         * 此方法是线程run方法
         */
        @Override
        public void run() {
            // TODO Auto-generated method stub
            while(isRunning) {

                draw();//刷新界面
                try {
                    Thread.sleep(20);//延时0.02s
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
                time += 1;
            }
        }

(2)	玩家飞机斜飞的实现：用四个布尔型的变量记录四个方向键的状态，按下时置为true，松开是置为false，而当为true时玩家飞机进行移动；
private boolean isPress01, isPress02, isPress03, isPress04;   //记录按键状态
void adapter(Canvas c) {

        c.addKeyListener(new KeyAdapter() {

            @Override
            public void keyPressed(KeyEvent e) {
                // TODO Auto-generated method stub

                int key = e.getKeyCode();
                if(key == KeyEvent.VK_UP)
                    isPress01 = true;

                if(key == KeyEvent.VK_DOWN)
                    isPress02 = true;

                if(key == KeyEvent.VK_LEFT)
                    isPress03 = true;

                if(key == KeyEvent.VK_RIGHT)
                    isPress04 = true;

            }

            @Override
            public void keyReleased(KeyEvent e) {
                // TODO Auto-generated method stub
                int key = e.getKeyCode();
                if(key == KeyEvent.VK_UP)
                    isPress01 = false;

                if(key == KeyEvent.VK_DOWN)
                    isPress02 = false;

                if(key == KeyEvent.VK_LEFT)
                    isPress03 = false;

                if(key == KeyEvent.VK_RIGHT)
                    isPress04 = false;
            }

        });

    }

(3)	敌机的实现：敌机以一个大小为八的一维数组存储，循环出现在地图视野中，每一架敌机的初始y坐标每一次循环都是一样的，x坐标每一次都是随机的，当敌机被打到或者超过地图下方时会重置坐标；
e_array = new Enemy[8];//创建敌机数组

//初始化敌机数组
            for(int i = 0; i < e_array.length; i++)
                e_array[i] = new Enemy((-i-1)*ENEMY_SIZE-BULLET_HEIGHT);
                
 /**
     * 绘制敌机方法
     * @param g
     * @param c
     */
    void drawEnemy(Graphics g, Canvas c, int i) {
        if(stayed)
            g.drawImage(ememyPic[i], enemy_x, enemy_y, GamePanel.ENEMY_SIZE, GamePanel.ENEMY_SIZE, c);//绘制敌机
        else if(id == 0) {
            b = new Break(enemy_x, enemy_y);
            b.enemy_break(g, c, id);
            id++;
        }

        if(b != null && id != 0)
            if(id == 29){
                b.enemy_break(g, c, id);
                id = 0;
            } else {
                b.enemy_break(g, c, id);
                id++;
            }
    }
    
(4)	玩家飞机子弹的实现：玩家飞机子弹对象用一个ArrayList存储，在固定时间间隔后绘制一个子弹对象并加入数组，当子弹超过地图时从数组中移除子弹对象；
if(mp.stayed) {
                //创建玩家飞机子弹对象
                b = new Bullet((int)(mp.getX_Y().getX())+PLANE_SIZE/2-BULLET_WIDTH/2, (int)(mp.getX_Y().getY())-BULLET_HEIGHT);
                b.drawBullet_1(array, gBuffer, this);//子弹绘制
                for(int i = 0; i<array.size() && array.size()>1; i++) {
                    array.get(i).drawBullet(gBuffer, this, 1);//子弹绘制
                }
            } else if(mp.id == 30){
                jb1.setEnabled(false);//设置暂停\开始按钮为不可用
                isRunning = false;//设置线程不循环
                new Dialog(m, 1);//打开对话框
            }

（5）敌机子弹的实现：所有敌机子弹对象都是用一个ArrayList存储，但每一个子弹对象依赖于每一架敌机，在循环绘制敌机时绘制每一架敌机的子弹；
            //绘制敌机、敌机子弹
            for(int i = 0; i < e_array.length; i++)
            {
                e_array[i].drawEnemy(gBuffer, this, i%5);
                if(e_array[i].stayed && e_array[i].getX_Y().getY()>0) {
                    int t = (int)(e_array[i].getX_Y().getY())+ENEMY_SIZE;
                    if(t < MAP_HEIGHT) {
                        //创建敌机子弹对象
                        b = new Bullet((int)(e_array[i].getX_Y().getX())+ENEMY_SIZE/2-BULLET_WIDTH/2, t);
                        b.drawBullet_2(array1, gBuffer, this, i);//子弹绘制
                    }

                }
                for(int j = 0; j<array1.size(); j++) {
                    array1.get(j).drawBullet(gBuffer, this, 2);//子弹绘制
                }
            }

（6）	Boss的实现：在一定的时间过后，敌机由于不满足if语句的时间已经超过将不再出现，而会出现boss，boss的折线运动是通过一个switch语句块实现的，由一个int类型的position变量作为运动方向的判断；
/**
     * 绘制boss
     * @param g
     * @param c
     */
    void drawBoss(Graphics g, Canvas c) {
        if(stayed)
            g.drawImage(ima, bossPlane_x, bossPlane_y, GamePanel.BOSS_WIDTH, GamePanel.BOSS_HEIGHT, c);
        else if(id == 0) {
            b = new Break(bossPlane_x, bossPlane_y);
            b.boss_break(g, c, id);
            id++;
        } else {
            b.boss_break(g, c, id);
            id++;
        }
    }
    /**
     * 控制boss移动
     */
    void bossMove() {
        if(bossPlane_y < 80)
            bossPlane_y += step;
        else
            switch (position) {//让飞机曲线运动
                case 0://向右下移
                    bossPlane_y += step;
                    bossPlane_x += step;
                    if(bossPlane_x == GamePanel.MAP_WIDTH-GamePanel.BOSS_WIDTH)
                        position++;
                    break;
                case 1://向左下移
                    if((point/700)%2 == 0) {
                        bossPlane_y += 3;
                        bossPlane_x -= step;
                    } else {
                        bossPlane_y += 2;
                        bossPlane_x -= 3;
                    }
                    if(bossPlane_y >= GamePanel.MAP_HEIGHT-GamePanel.BOSS_HEIGHT)
                        position++;
                    break;
                case 2://向左上移
                    if((point/700)%2 == 0) {
                        bossPlane_y -= step;
                        bossPlane_x -= step;
                    } else {
                        bossPlane_y -= 3;
                        bossPlane_x -= step;
                    }
                    if(bossPlane_x <= 0)
                        position++;
                    break;
                case 3://向右上移
                    bossPlane_x += step;
                    if(bossPlane_x == 175)
                        position = -1;
                    break;
                default://向右移动
                    point++;
                    if(point%300 == 0)
                        position = 0;
                    break;
            }
    }
 
 （7）	Boss散射子弹的实现：用大小为五的子弹数组存储boss发射的五颗发散的子弹，再用ArrayList数组存储子弹数组对象，子弹发射方向用一个switch语句块根据子弹数组的角标0~4来确定；
 /**
     * boss子弹移动方法
     * @param i
     */
    void bulletMove2(ArrayList<Bullet[]> arr, int i, int j) {
        if(stayed)
            switch (j) {
                case 0:
                    bullet_x -= 2;
                    bullet_y += 2;
                    break;
                case 1:
                    bullet_x -= 1;
                    bullet_y += 2;
                    break;
                case 2:
                    bullet_y += 2;
                    break;
                case 3:
                    bullet_x += 1;
                    bullet_y += 2;
                    break;
                case 4:
                    bullet_x += 2;
                    bullet_y += 2;
                    break;
                default:
                    break;
            }
        else
            arr.get(i)[j] = null;
    }
    
   （8）	血条的实现：玩家飞机和boss的生命用一个静态的int型变量记录，血条通过21张代表不同血量的图片根据记录生命的变量的大小绘制不同的图片，boss血条的绘制的坐标根据boss飞机的坐标确定，以保证血条在boss的上方；
   if(live1 >= 0 && bp.stayed) {
                //绘制boss对象
                x = Toolkit.getDefaultToolkit().getImage(getClass().getResource("/images/xue_" + ((2000-live1)/100+1) + ".png"));
                gBuffer.drawImage(x, (int)(bp.getX_Y().getX()), (int)(bp.getX_Y().getY()-10), 250, 10, this);
            }

            if(live >= 0) {
                //绘制玩家飞机对象
                x = Toolkit.getDefaultToolkit().getImage(getClass().getResource("/images/xue_" + ((100-live)/5+1) + ".png"));
                gBuffer.drawImage(x, 7, 7, 200, 20, this);
            }

 （9）	暂停的实现：根据isRunning为true或者false，可判断另一个线程中的循环是否进行，如果不进行，那么游戏地图将不会重绘，达到暂停效果；
 /**
     * 此方法用于执行暂停与开始
     */
    private void stop_start() {
        if(isRunning == true)
        {
            isRunning = false;//设置线程不循环
            jb1.setText("开始(P)");//改变按钮文字

        } else {
            isRunning = true;//设置线程循环
            Thread d = new Thread((Runnable) jp);//创建线程
            d.start();//开启线程
            jb1.setText("暂停(P)");//改变按钮文字
        }
    }
    
   （10）	重新开始按钮的功能：当按下重新开始按钮，程序将会重新创建一个绘制地图的内部类（MapPanel）对象，同时置isRunning为false，并且还原一些参数，如：分数、血量等；
   jb2.addActionListener(new ActionListener() {

            @Override
            public void actionPerformed(ActionEvent e) {
                // TODO Auto-generated method stub
                //刷新游戏界面
                remove(jp);
                jp = new MapPanel();
                jp.setBounds(200, 0, 600, 600);
                add(jp);
                //让游戏暂停
                isRunning = false;
                jb1.setText("开始(P)");//改变按钮文字
                Bullet.before_time = System.currentTimeMillis();

                jb1.setEnabled(true);//设置暂停\开始按钮为可按
                sum = 0;//设置分数显示为0

                live = 100;//设置生命值100
                live1 = 2000;//设置boss生命值2000

                time = 0;//赋予时间

            }
        });
        
   （11）	声音的实现：游戏声音的控制封装为一个PlaySound类，有根据传入的路径打开音频文件、建立音频行、通过调用音频行对象的start方法开始播放、通过调用音频行对象的stop方法暂停播放、通过调用音频行对象的loop方法重复播放等几个方法，当需要控制声音时调用相应方法。
    /**
     * 打开声音文件方法
     * @param s
     */
    void open(String s) {
        file = new File(s);//音频文件对象
        try {
            stream = AudioSystem.getAudioInputStream(file);//音频输入流对象
            format = stream.getFormat();//音频格式对象
        } catch (UnsupportedAudioFileException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
    
    /**
     * 建立播放音频的音频行
     */
    void play() {
        info = new DataLine.Info(Clip.class, format);//音频行信息对话框
        try {
            clip = (Clip) AudioSystem.getLine(info);//音频行对话框
            clip.open(stream);//将音频数据读入音频行
        } catch (LineUnavailableException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
    
    /**
     * 停止播放
     */
    void stop() {
        clip.stop();//暂停音频播放
    }

    /**
     * 开始播放
     */
    void start() {
        clip.start();//播放音频文件
    }

    /**
     * 回放背景音乐设置
     */
    void loop() {
        clip.loop(20);//回放
    }
}

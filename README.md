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

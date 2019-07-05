      
#  C 语言实现Atari经典街机游戏《飞天蜈蚣》

[![LICENSE](https://img.shields.io/cocoapods/l/AFNetworking.svg)](https://github.com/Hephaest/AtariCentipedeGame/blob/master/LICENSE)
[![OS](https://img.shields.io/badge/OS-Ubuntu16.04-lightgrey.svg)](http://releases.ubuntu.com/16.04/)
[![Library](https://img.shields.io/badge/implementation-ncurses-blue.svg)](http://www.tldp.org/HOWTO/NCURSES-Programming-HOWTO/)

中文介绍 | [English Introduction](README.md)

<p align=center>
<img src="https://github.com/Hephaest/AtariCentipedeGame/blob/master/images/Level2-demo.gif" width = "800"/>
</p>

这是一个用C语言实现的《飞天蜈蚣》的“复刻”，由于Linux操作系统下的图形库不够绘制绚丽的画面，因此在本程序中仅用**简单的符号**来代表不同的游戏角色.<br/>
希望您喜欢这个版本的飞天蜈蚣，如有任何疑问或建议欢迎在 **issue** 处留言. :)<br/>
建议您最好通读本介绍后再运行`magic.c`程序.<br/>
访问本油管链接可以阿达雅游戏的视频: https://www.youtube.com/watch?v=dxoK8hosHjA&t=1s!<br/>

# 基本介绍
**注意**: 本程序需要引入 `ncurses` 库, 这个库只存在于 Unix 的操作系统. 在开始运行之前,建议您通过以下 Debian/Ubuntu Linux 的命令行安装需要的库:
```C
sudo apt−get install libncurses5−dev libncursesw5−dev
//libncurses5−dev : Developer's libraries for ncurses
//libncursesw5−dev : Developer's libraries for ncursesw
```
## 问题重申
基于原始的雅达利游戏, 我重写了部分规则. 为了使我的程序画面更加稳定和流程，我做了以下必要的前提条件.
### 游戏规则
蝎子是游戏里另一个一意孤行的角色. 相比于蜘蛛, 它就完全是个坏蛋, 它虽然只能向左或向右走但是一旦它碰到玩家玩家就会失去一条生命.<br>

**蜈蚣**<br>
蜈蚣是玩家最主要要消灭的敌人. 如果没有任何障碍物, 蜈蚣只会一行接着一行地往下走.每次玩家击中蜈蚣的部位都会变成一个蘑菇. 如果击中的部位既不在头也不在尾, 蜈蚣就会自己撕裂成两部分, 获得新的头部并向上或向下接近玩家. 除此之外, 如果蜈蚣遇到蘑菇或者墙壁时, 它会拐弯下移. 尽量让自己远离蜈蚣，如果被它碰到的话也会失去生命哦!<br>

为了让上述规则更易懂, 我创建了如下关系表格:

|       撞击关系       |                       玩家                       |                        子弹                         |                           蘑菇                           |       蜈蚣        |
| :-------------: | :------------------------------------------------: | :----------------------------------------------------: | :----------------------------------------------------------: | :--------------------: |
|   **蜘蛛**    |             玩家死亡, 失误一次生命              |               蜘蛛死亡, 获得600分               |      蘑菇消失, 蘑菇总数 - 1      |         不考虑         |
| **蝎子** |             玩家死亡, 失误一次生命              |            蝎子死亡, 获得600分             |                            不考虑                            |         不考虑         |
|  **蜈蚣**  |             玩家死亡, 失误一次生命              | 击中头部给100分，其余情况给10分 | 蜈蚣掉头 | 其中一只蜈蚣掉头 |
|  **蘑菇**   | 蘑菇消失, 蘑菇总数 - 1 |                    4 次成功射击后获得1分                    |                            不考虑                            |  不考虑  |

除此以外, 我在游戏中设置了不同关卡. 每个关卡初始的蜈蚣身节是相等的. 但随着蘑菇数量的增多, 玩家很难能击中蜈蚣. 更不用说蝎子和蜘蛛了.

### 基本猜想
我在运行程序的时候遇到许多非程序性问题. 因此, 我罗列出必要的假设:
- 在每个关卡最开始的时候, 如果蘑菇所在位置和蜈蚣初始位置重合, 玩家可以认为该蘑菇会被蜈蚣吃掉.
- 蜘蛛只会出现在游戏窗口的下端. 相反的, 蝎子只会出现在游戏窗口的上方.
- 玩家的操控范围不能超过游戏窗口的边界. 除此之外, 玩家不能穿越蘑菇但子弹可以.
- 蜈蚣被射中的身节会变成蘑菇, 这些新生成的蘑菇会随着失败重玩或闯关成功而存在除非被吃掉或击中.

# 游戏流程图
<p align=center>
<img src="https://github.com/Hephaest/AtariCentipedeGame/blob/master/images/flow.png"/>
</p>
下图详细地介绍整个游戏的流程, 包括碰撞结果, 分数更新和角色移动等. 在这些过程中, 每次计算机从 I/O 获得运行顺序并决定是满足碰撞条件, 然后, 再创新画布并更新分数.<br>
如果您仍然不熟悉游戏流程，可访问下方的油管链接观看游戏通关全过程:<br>
https://www.youtube.com/watch?v=iQRfnJUAYQs<br>
或者查看原始的流程图:<br>
https://www.processon.com/view/link/5b040ea4e4b05f5d6b641243

# 函数实现

## 开始菜单
首先, 我需要创建一个主菜单来简单介绍游戏规则. 这个滚动菜单需要给用户提供两种选择: 开始游戏或退出游戏.
```C
/* 这些选项将在介绍菜单里显示*/
char *startMenu[] = {
        "Start", "Exit",
};

/* 介绍菜单里的文字内容*/
char *readme[] = {
        "*Centipede Game*",
        " ",
        "This game is written by Hephaest, Lancaster University.",
        "Before the game, you'd better recognize these following characters.",
        " ",
        "*Character*",
        " ",
        "_^_",
        "|H|  Master",
        "---",
        " ",
        "This role is for you! you can move it by pressing keyboard:",
        "KEY_LEFT, KEY_RIGHT, KEY_UP and KEY_DOWN.",
        "Besides, you can fire bullets by pressing the blank space key.",
        " ",
        "*********O Centipede",
        " ",
        "This is the major character you need to shoot.",
        "Once it hits you, you lose your life.",
        "You only have 4 lives, be careful!",
        " ",
        "& Mushroom",
        " ",
        "This is a barrier, you cannot walk through it unless you destory it.",
        "Once you destory one of them, you will lost 4 bullets.",
        "Don't worry, you have tons of bullets.",
        " ",
        "_$ Sea monster",
        " ",
        "This character walks line by line. You can shoot it and get bonus.",
        "However, once it hits you, you lose your life.",
        " ",
        "^W^ Spider",
        " ",
        "This character crawls all the time.",
        "Spider could either help you by eating mushrooms or hitting you.",
        " ",
        "*Tips*",
        " ",
        "Press \'P\' or \'p\' to pause.",
        "Press \'C\' or \'c\' to continue.",
        "Press \'Q\' or \'q\' to quit or replay.",
        " ",
        "All in all, be careful! Hope you enjoy the game!",
        " ",
};

/* 下列选项将出现在退出菜单 */
char *quitMenu[] = {
        "No",
        "Yes",
        "Replay",
};
```
在此之后, 我需要创建一个窗口来显示这个菜单. 因为我没有找到滚动菜单的UI框架, 只要再每行文字里追加滚动菜单属性.
```C
/* 此功能是简介菜单里用户指针使用 */
void start()
{
    ITEM **start_items;
    int c;
    MENU *menu;
    WINDOW *start_menu;
    int n_startMenu, item, n_readme;

    /* Curses initialization */
    initscr();
    start_color();
    cbreak();
    noecho();
    init_pair(1, COLOR_WHITE, COLOR_BLACK);
    init_pair(2, COLOR_RED, COLOR_YELLOW);

    /* 创建介绍菜单选项.
     * 以下操作主要旨在实现以下要求：
     * 1. Readme 只是用来显示文字的.
     * 2. 高亮所有在开始菜单里的选项.
     */
    n_startMenu = ARRAY_SIZE(startMenu);
    n_readme = ARRAY_SIZE(readme);
    start_items = (ITEM **)calloc(n_startMenu+n_readme+1, sizeof(ITEM *));
    for(item = 0; item < n_readme; item++)
    {
        start_items[item] = new_item(readme[item], readme[item]);
        item_opts_off(start_items[item], O_SELECTABLE);
        set_item_userptr(start_items[item], Click);
    }
    int cho = 0;
    for(;item < n_readme + n_startMenu; item++)
    {
        start_items[item] = new_item(startMenu[cho], startMenu[cho]);
        /* Set the user pointer, even in readme*/
        set_item_userptr(start_items[item], Click);
        cho++;
    }
    /* 设置最后一个为 NULL */
    start_items[n_readme + n_startMenu] = (ITEM *)NULL;
    /* 创建菜单 */
    menu = new_menu((ITEM **)start_items);

    /* 这些颜色设置将应用于开始选项*/
    set_menu_fore(menu, COLOR_PAIR(2));
    set_menu_back(menu, COLOR_PAIR(1));
    set_menu_grey(menu, COLOR_PAIR(1));
    menu_opts_off(menu, O_SHOWDESC);

    /* 创建窗户并让他位于屏幕中央 */
    int max_y, max_x;
    getmaxyx(stdscr, max_y, max_x);
    int x_position = (max_x-box_length)/2;
    int y_position = (max_y-box_width)/2;
    start_menu = newwin(box_width, box_length, y_position, x_position);
    keypad(start_menu, TRUE);//To listen keyboard

    /* 设置主窗口和子窗口 */
    set_menu_win(menu, start_menu);
    set_menu_sub(menu, derwin(start_menu, box_width-1, box_length-2, 1, 1));
    set_menu_format(menu, 18, 1);// Set 18 lines inside the box and one choices each line

    /* 创建边界框 */
    box(start_menu, 0, 0);
    refresh();

    /* 发布菜单 */
    post_menu(menu);
    wrefresh(start_menu);

    while((c = wgetch(start_menu)) != KEY_END)
    {
        switch(c)
        {	case KEY_DOWN:
                menu_driver(menu, REQ_DOWN_ITEM);
                break;
            case KEY_UP:
                menu_driver(menu, REQ_UP_ITEM);
                break;
            case KEY_NPAGE:
                menu_driver(menu, REQ_SCR_DPAGE);
                break;
            case KEY_PPAGE:
                menu_driver(menu, REQ_SCR_UPAGE);
                break;
            case 10: /* This is Enter key*/
            {
                /*To point to current choice*/
                ITEM *cur;
                void (*p)(char *);
                cur = current_item(menu);
                p = item_userptr(cur);
                p((char *)item_name(cur));
                pos_menu_cursor(menu);
                break;
            }
        }

        wrefresh(start_menu);
    }

    /* 清除菜单并清除缓存 */
    unpost_menu(menu);
    for(int i = 0; i < n_startMenu + n_readme; ++i)
        free_item(start_items[i]);
    free_menu(menu);
    /* 退出当前窗口并等待下一个窗口*/
    endwin();
}
```
主菜单被分为2部分: 第一部分是文字(只是用来显示, 就算点击了也无任何响应). 在第二部分, 设置用户指针允许玩家做出选择. 为了提醒玩家确定自己的选择, 我高亮每个选项的背景色. 
```C
/* 过滤选择 */
void Click(char *choice)
{
    if(choice == "Exit")
    {
        endwin();
        exit(0);
    }
    if(choice == "Start") {
        clear();
        refresh();
        Initialization();
        GameInterface();
    }
}
```
以上代码的运行结果如下图所示.
<p align=center>
<img src="https://github.com/Hephaest/AtariCentipedeGame/blob/master/images/HomePage.png"/>
</p>

## 角色扮演

**蘑菇** 被随机地放置在整个游戏窗口, 可以使用循环语句查找位置重叠的蘑菇并给它们重新设置新的位置.

```C
/*
 * 创建蘑菇
 * 以下操作主要旨在实现以下要求：
 * 1. 蘑菇的位置应随机生成.
 * 2. 第一行和后三行被蜈蚣和玩家预先占有.
 */
void MushroomProduce(int y, int x)
{
    int i = 0;
    mushLength = 20;
    srand(time(NULL));
    do {
        mushroom[i].x = rand() % (x - 3) + 1;
        mushroom[i].y =rand() % (y - 4) + 1;
        for (int j = 0; j < mushLength; j++) {
            if (mushroom[i].x == mushroom[j].x && mushroom[i].y == mushroom[j].y && i != j)
            {
                mushroom[i].x = rand() % (x - 2);
                mushroom[i].y =rand() % (y - 4) + 1;
            }
        }
        i++;
    } while (i < mushLength); /* 直到没有重叠 */
    for(int i = 0; i < mushLength; i++) mushroom[i].mush_record=4;
}
```

**玩家** 通过使用 `getmaxyx()` 函数来放置在游戏窗口下方中央的位置.

```C
/*
 * 玩家创建
 * 以下操作主要旨在实现以下要求：
 * 1. 确保玩家在游戏窗口底端的中央位置.
 */
void MasterProduce(WINDOW *win)
{
    int win_y;
    int win_x;
    getmaxyx(win, win_y, win_x);
    master_1_y = win_y - 3;
    master_1_x = (win_x - master_length) / 2;
    master_2_y = win_y - 2;
    master_2_x = (win_x - master_length) / 2;
    master_3_y = win_y - 1;
    master_3_x = (win_x - master_length) / 2;
    curr_bullet_y = master_1_y;
    curr_bullet_x = master_1_x + 1;
    /* 绘制窗口 */
    mvwprintw(win, master_1_y, master_1_x, "_");
    mvwprintw(win, master_1_y, master_1_x + 1, "^");
    mvwprintw(win, master_1_y, master_1_x + 2, "_");
    mvwprintw(win, master_2_y, master_2_x, "|") ;
    mvwprintw(win, master_2_y, master_2_x + 1, "H");
    mvwprintw(win, master_2_y, master_2_x + 2, "|");
    mvwprintw(win, master_3_y, master_3_x, "-");
    mvwprintw(win, master_3_y, master_3_x + 1, "-");
    mvwprintw(win, master_3_y, master_3_x + 2, "-");
}
```

**蜈蚣** 通过 `getmaxyx()` 方法放置在游戏窗口上方的中央位置并使用 `positions(x,y)` 从尾到头赋值.

```C
/*
 * 创建蜈蚣
 * 从尾部开始
 */
void CentipedeProduce(int x)
{
    int i;
    Length = 10;
    int start_x = (x - Length) / 2;
    for (i = Length - 1; i >= 0; i--) {
        Centipede[i].x = start_x;
        Centipede[i].y = 0;
        start_x++;
    }
    for(int i = 0; i < Length; i++)
    {
        for(int j = 0; j < mushLength; j++)
        {
            if(Centipede[i].x == mushroom[j].x && Centipede[i].y == mushroom[j].y)
            {
                j++;
                while (j < mushLength) {
                    mushroom[j - 1].x = mushroom[j].x;
                    mushroom[j - 1].y = mushroom[j].y;
                    mushroom[j - 1].mush_record = mushroom[j].mush_record;
                    j++;
                }
                mushLength -= 1;
            }
        }
        Centipede[i].head = -1;
        Centipede[i].Clear = -1;
        Centipede[i].x_direction = 1;
        Centipede[i].y_direction = 1;
    }
    Centipede[0].head = 0;
}
```
对于蜘蛛和蝎子而言, 然而, 它们并不总是出现在固定位置, 需要使用变量 `num` 来决定从游戏窗口左侧还是右侧出现. 除此之外, 可以使用另一个随机变量控制它们的出现与否 (详见下方源码).

```C
/*
 * 创建蝎子
 * 以下操作主要旨在实现以下要求：
 * 确保蝎子随机出现.
 */
void Sea_MonsterProduce(int y, int x)
{
    srand(time(NULL));
    int start_y = rand() % y/2+1;
    int num = rand() % 2;
    if(num == 0)
    {
        Sea_Monster[0].x = 1;
        Sea_Monster[0].y = start_y;
        Sea_Monster[1].x = 0;
        Sea_Monster[1].y = start_y;
        Sea_Monster[0].x_direction = 1;
    }
    if(num == 1)
    {
        Sea_Monster[0].x = x - 2;
        Sea_Monster[0].y = start_y;
        Sea_Monster[1].x = x - 1;
        Sea_Monster[1].y = start_y;
        Sea_Monster[0].x_direction = -1;
    }
}

/*
 * 创建蜘蛛
 * 以下操作主要旨在实现以下要求：
 * 确保蜘蛛随机出现.
 */
void SpiderProduce(int y, int x)
{
    int start_y = y - (rand() % y / 2 + 1);
    if(num == 1)
    {
        int start_x = 0;
        for(int i = 0; i < 3; i++)
        {
            Spider[i].x = start_x;
            Spider[i].y = start_y;
            start_x++;
            Spider[i].x_direction = 1;
            Spider[i].y_direction = 1;
        }
    }
    if(num == -1)
    {
        int start_x = x - 2;
        for(int i = 0; i < 3; i++)
        {
            Spider[i].x = start_x;
            Spider[i].y = start_y;
            start_x--;
            Spider[i].x_direction = -1;
            Spider[i].y_direction = 1;
        }
    }
}
```

随机函数:

```C
 /* 当蜈蚣在移动使，蝎子和蜘蛛也有随机出现的机会 */
    srand(time(NULL));
    int ran_sea = rand() % 10;
    if(ran_sea == 1) sea_appear=1;
    int ran_spider = rand() % 5;
    if(ran_spider == 1) spider_appear = 1;
```

## 移动的敌人
这些敌人包括蜘蛛, 蝎子和蜈蚣. 前两者比后者在处理上简单很多.

<p align=center>
<img src="https://github.com/Hephaest/AtariCentipedeGame/blob/master/images/monster.png" width = "500"/>
</p>

如上图所示, 蝎子是平行移动的, 并会撞上游戏窗口的边界, 计算机会给它一个初始位置并让它消失在游戏窗口. 而蜘蛛的爬行是很随意的, 无规律可言. 然而, 我使用了比 ***飞天蜈蚣*** 种更复杂的方法实现蜘蛛的爬行: 在 x 和 y 方向上的二维随机数. 但这也可能会造成蜘蛛在同一条轨迹上来回爬行 (详见下图) 但由于随机数 x 和 y 都是非负数因此蜘蛛最终会总窗口一端爬向窗口另一端.
<p align=center>
<img src="https://github.com/Hephaest/AtariCentipedeGame/blob/master/images/trace.png" width = "300"/>
</p>

除此以外, 我设置了蜘蛛 x 和 y 方向的边界条件以防它撞上自己的同伴蝎子. 另外, 我设置了布尔二值来延缓蜘蛛爬行的速度并让他依次从游戏窗口的左边或右边出现.

```C
/* 移动的蝎子.
 * 此功能为了实现蝎子的水平移动.
 */
void sea_monsterMove(int x)
{
        Sea_Monster[1].x = Sea_Monster[0].x;
        Sea_Monster[0].next_x = Sea_Monster[0].x + Sea_Monster[0].x_direction;
        /* 如果蝎子即将走到窗口边界, 使其消失 */
        if (Sea_Monster[0].next_x > x || Sea_Monster[0].next_x < 0)
        {
            Sea_Monster[0].x = -1;
            Sea_Monster[1].x = -1;
            sea_appear = 0;
        }
        else
        {
            Sea_Monster[0].x += Sea_Monster[0].x_direction;
        }
}

/* 移动的蜘蛛.
 * 此功能为了实现蜘蛛在 x 和 y 方向的移动.
 */
void SpiderMove(int x,int y)
{
    srand(time(NULL));
    int ran_y = y-(rand() % y / 2 + 3);
    int ran_x = rand() % 2;
    for(int j = 0; j < 3; j++)
    {
        Spider[j].next_x = Spider[j].x + ran_x*Spider[j].x_direction;
        Spider[j].next_y = Spider[j].y + Spider[j].y_direction;
        /* 如果蜘蛛即将走到窗口边界, 重设初始位置并使其消失 */
        if (Spider[j].next_x > x || Spider[j].next_x < 0)
        {
            Spider[0].x = -1;
            Spider[1].x = -1;
            Spider[2].x = -1;
            spider_appear = 0;
            num *= -1;
            break;
        }
        /* 如果蜘蛛将触碰游戏窗口的下边界, 更改 y 方向*/
        if(Spider[j].next_y > y - 1 || Spider[j].next_y < ran_y)
        {
            Spider[j].y_direction *= -1;
        }
            Spider[j].x += ran_x*Spider[j].x_direction;
            Spider[j].y += Spider[j].y_direction;
    }
}
```

就蝎子而言, 它是本游戏中最复杂的情况. 我将我能想到的所有情况都在下方列出. 我主要使用方法是结构体和数组. 因为碰撞每次都需要考虑每个身节的变化情况, 因此数组更容易帮我定位到指定身节. 每一次移动, 只需要处理蜈蚣头部运动情况, 并让它的身节跟着头部运动即可. 最后, 重刷画布. 具体的操作已在下方源码中列出.

<p align=center>
<img src="https://github.com/Hephaest/AtariCentipedeGame/blob/master/images/HitMush.png" width = "400"/>
</p>

<p align=center>
<img src="https://github.com/Hephaest/AtariCentipedeGame/blob/master/images/HitWall.png" width = "400"/>
</p>

```C
/*
 * 此功能用于确定蜈蚣的每个身节应该到达的位置.
 * 一旦蜈蚣开始移动, 需要对它的运动情况进行分开讨论.
 */
void CentipedeMove(int x, int y)
{
    for(int i = 0; i < Length; i++)
    {
        int skip=0;//This is used to mark
        /* 仅移动尚未射中的蜈蚣的身节 */
        if(Centipede[i].head >=0 && Centipede[i].Clear < 0)
        {
            int j = i;
            int k = i + 1;
            /* 对于多个蜈蚣，识别他们的头部，并使他们自己的身体跟随他们 */
            while(k <= Length - 1 && Centipede[k].head < 0 && Centipede[k].Clear < 0)
            {
                j++;
                k++;
            }
            while(j > i)
            {
                Centipede[j].x = Centipede[j - 1].x;
                Centipede[j].y = Centipede[j - 1].y;
                Centipede[j].x_direction = Centipede[j-1].x_direction;
                j--;
            }
            Centipede[i].next_x = Centipede[i].x + Centipede[i].x_direction;
            Centipede[i].next_y = Centipede[i].y + Centipede[i].y_direction;
            /* 如果蜈蚣即将撞上游戏窗口左右边界, 它需要下移并掉头. 然而:
             * 1. 如果有另一条蜈蚣正在它下方, 此蜈蚣只能原路返回.
             * 2. 如果蜈蚣下移时会撞上蘑菇. 那么, 蜈蚣需要躲避蘑菇.
             * 3. 如果蜈蚣下移后的下一秒会撞上蘑菇, 它就无法掉头只能再继续下移一格.
             * 4. 如果下移后蜈蚣即将撞上游戏窗口的下边界, 蜈蚣需要上移掉头.
             */
            if (Centipede[i].next_x > x || Centipede[i].next_x < 0)
            {
                /* 针对情况1 */
                for(int k = 0; k < Length; k++)
                {
                    if(Centipede[k].Clear < 0 && (k < i || k > j))
                    {
                        if(Centipede[i].next_y == Centipede[k].y && Centipede[i].x == Centipede[k].x)
                        {
                            Centipede[i].x_direction *= -1;
                            skip = 1;
                        }
                    }
                }

                for(int j = 0; j < mushLength; j++)
                {
                    /* 针对情况2 */
                    if(Centipede[i].next_y == mushroom[j].y && Centipede[i].x == mushroom[j].x)
                    {
                        Centipede[i].x_direction *= -1;
                        Centipede[i].y += 1;
                        skip=1;
                        Centipede[i].x += Centipede[i].x_direction;
                    }
                    /* 针对情况3 */
                    if(Centipede[i].next_y == mushroom[j].y && Centipede[i].x - 1 == mushroom[j].x || Centipede[i].next_y == mushroom[j].y && Centipede[i].x + 1 == mushroom[j].x)
                    {
                        Centipede[i].y += 1;
                        skip = 1;
                    }
                }
                /* 针对情况4 */
                if (Centipede[i].next_y > y)
                {
                    Centipede[i].y -= 3;
                    Centipede[i].x_direction *= -1;
                    skip = 1;
                }
                /* 默认情况, 只需要下移掉头 */
                if(!skip)
                {
                    Centipede[i].x_direction *= -1;
                    Centipede[i].y += 1;
                }
            }
            else
            {
                /* 如果蜈蚣的头部即将撞上蘑菇, 它将会下移掉头. 然而:
                 * 5. 如果它正下方是另一条蜈蚣, 该蜈蚣只能原路返回.
                 * 6. 如果下移后会撞上蘑菇. 它需要躲避蘑菇.
                 */
                for(int j = 0; j < mushLength; j++)
                {
                    if(Centipede[i].next_x == mushroom[j].x && Centipede[i].y == mushroom[j].y)
                    {
                        /* 针对情况5 */
                        for(int k = 0; k < Length; k++)
                        {
                            if(Centipede[k].Clear < 0 && (k < i || k > j))
                            {
                                if(Centipede[i].next_y == Centipede[k].y && Centipede[i].x == Centipede[k].x)
                                {
                                    Centipede[i].x_direction *= -1;
                                    skip = 1;
                                }
                            }
                        }
                        /* 针对情况6 */
                        for(int k = 0; k < mushLength; k++)
                        {
                            if(Centipede[i].next_y == mushroom[k].y && Centipede[i].x == mushroom[k].x && j!=k)
                            {
                                Centipede[i].x_direction *= -1;
                                Centipede[i].y += 1;
                                skip = 1;
                                Centipede[i].x += Centipede[i].x_direction;
                            }
                        }
                        /* 默认情况, 只需要下移和头 */
                        if(!skip)
                        {
                            Centipede[i].x_direction *= -1;
                            Centipede[i].y += 1;
                            skip = 1;
                        }
                    }
                }
                /* 针对情况 7
                 * 如果该蜈蚣和另一条蜈蚣相向而行, 那么:
                 * 该蜈蚣接受妥协, 向下移动并掉头.
                 */
                for(int k = 0; k < Length; k++)
                {
                    if(Centipede[k].Clear < 0 && (k < i || k > j))
                    {
                        if(Centipede[i].next_x == Centipede[k].x && Centipede[i].y == Centipede[k].y)
                        {
                                Centipede[i].x_direction *= -1;
                                Centipede[i].y += 1;
                                skip = 1;
                        }
                     }
                }
                /* 如果都不属于上述情况, 只需要让蜈蚣继续沿 x 方向移动 */
                if(skip==0) Centipede[i].x += Centipede[i].x_direction;
            }
        }
    }
}
```
## 移动的玩家

我使用 `getch()` 函数来监听键盘输入. 然而, 在同步 I/O 中监听意味着计算机无法调度程序的其他部分运行. 因此必须使用多线程来监听键盘输入. 这部分没有什么需要探讨的实现原理, 只需要记住玩家是不能穿越游戏窗口的边界和蘑菇的. 以下是源码部分:
```C
/*
 * 此功能用于始终从键盘接收玩家输入！
 */
void * waitForKey() {
    while (1) {
        usleep(10);//以防宏块
        ch = getch();
    }
}

/*
 * 键盘监听器
 * 以下操作主要实现以下要求：
 * 1. 玩家可以上下左右移动.
 * 2. 玩家按下空格键即可射击.
 * 3. 如果玩家触碰蘑菇, 玩家暂停移动.
 * 4. 如果玩家触碰游戏窗口边界, 玩家暂停移动.
 * 5. 当子弹被触发时, 子弹应从玩家当前位置射出.
 */
void getOrder(WINDOW *win, int x, int y)
{
    int skip = 0;
    switch(ch)
    {
        case KEY_LEFT:
            for(int i = 0; i < mushLength; i++)
            {
                if(master_1_x == 0 || (master_1_y == mushroom[i].y && master_1_x == mushroom[i].x + 1) || (master_2_y == mushroom[i].y && master_2_x == mushroom[i].x + 1) || (master_3_y == mushroom[i].y && master_3_x == mushroom[i].x + 1))//on the  left hand of master
                {
                    // 略过
                    skip = 1;
                }
            }
            if(!skip)
            {
                master_1_x -= 1;
                master_2_x -= 1;
                master_3_x -= 1;
                curr_bullet_y = master_1_y;
                curr_bullet_x = master_1_x + 1;
            }
            break;
        case KEY_RIGHT:
            for(int i = 0; i < mushLength; i++)
            {
                if ((master_1_x + 2) == x - 1 || (master_1_y == mushroom[i].y && master_1_x + 2 == mushroom[i].x - 1) ||
                    (master_2_y == mushroom[i].y && master_2_x + 2 == mushroom[i].x - 1) ||
                    (master_3_y == mushroom[i].y && master_3_x + 2 == mushroom[i].x - 1))
                {
                    // 略过
                    skip=1;
                }
            }
            if(!skip)
            {
                master_1_x += 1;
                master_2_x += 1;
                master_3_x += 1;
                curr_bullet_y = master_1_y;
                curr_bullet_x = master_1_x + 1;
            }
            break;
        case KEY_UP:
            for(int i = 0; i < mushLength; i++)
            {
                if (master_1_y == 0 || (master_1_y == mushroom[i].y + 1 && master_1_x == mushroom[i].x) || (master_1_y == mushroom[i].y + 1 && master_1_x + 1 == mushroom[i].x) || (master_1_y == mushroom[i].y + 1 && master_1_x + 2 == mushroom[i].x))
                {
                    // 略过
                    skip=1;
                }
            }
            if(!skip)
            {
                master_1_y -= 1;
                master_2_y -= 1;
                master_3_y -= 1;
                curr_bullet_y = master_1_y;
                curr_bullet_x = master_1_x + 1;
            }
            break;
        case KEY_DOWN:
            for(int i = 0; i < mushLength; i++)
            {
                if (master_3_y == y - 1 || (master_3_y == mushroom[i].y - 1 && master_3_x == mushroom[i].x) || (master_3_y == mushroom[i].y - 1 && master_3_x + 1 == mushroom[i].x) || (master_3_y == mushroom[i].y - 1 && master_3_x + 2 == mushroom[i].x))
                {
                    // 略过
                    skip = 1;
                }
            }
            if(!skip)
            {
                master_1_y += 1;
                master_2_y += 1;
                master_3_y += 1;
                curr_bullet_y = master_1_y;
                curr_bullet_x = master_1_x + 1;
            }
            break;
        case ' ':
            bullet_x = curr_bullet_x;
            bullet_y = curr_bullet_y;
            Fire = 1;
            break;
    }
    ch = 0;
    mvwprintw(win, master_1_y, master_1_x, "_");
    mvwprintw(win, master_1_y, master_1_x + 1, "^");
    mvwprintw(win, master_1_y, master_1_x + 2, "_");
    mvwprintw(win, master_2_y, master_2_x, "|");
    mvwprintw(win, master_2_y, master_2_x + 1, "H");
    mvwprintw(win, master_2_y, master_2_x + 2, "|");
    mvwprintw(win, master_3_y, master_3_x , "-");
    mvwprintw(win, master_3_y, master_3_x + 1, "-");
    mvwprintw(win, master_3_y, master_3_x + 2, "-");
}
```

## 菜单操作

我在游戏界面创建了三个选项. 如果玩家按下 **P** 或 **p**, 该按钮将消失. 所有的操作都会被暂停并且 **Continue** 按钮出现. 相反地, 如果玩家按下 **C** 或 **c**, 该按钮将消失. 所有被中断的操作继续并且 **Pause** 按钮出现. 除此之外, 玩家可以随时退出游戏窗口, 只需要按下 **Q** 或者 **q** 并选择 **Yes**.

<p align="center"><img src="https://github.com/Hephaest/AtariCentipedeGame/blob/master/images/screens.png" width = "500"></p>

<p align="center"><img src="https://github.com/Hephaest/AtariCentipedeGame/blob/master/images/quit.png" width = "250"></p>

```C
/* 这个结构体为了实现 "Pause" 和 "Continue" 按钮的隐藏 */
typedef struct PANEL_HIDE {
    int hide;
}Panel;

/*
 * 事件监听器
 * 以下操作主要实现以下要求:
 * 1. 如果玩家按下 "Continue", 该按钮消失并显示 "Pause" 按钮.
 * 2. 如果玩家按下 "Pause", 该按钮消失并显示t "Continue" 按钮.
 */
void getInt(PANEL *Con, PANEL *Pau, int x, int y, WINDOW* win)
{
    Panel *temp;
    switch(ch)
    {
        case 'c':
        case 'C':
            temp = (Panel *)panel_userptr(Con);
            hide_panel(Con);
            temp->hide = TRUE;
            if(temp->hide == TRUE)
            {
                temp = (Panel *) panel_userptr(Pau);
                show_panel(Pau);
                temp->hide = FALSE;
            }
            stop = 0;
            interrupt_end = time(NULL);
            break;
        case 'p':
        case 'P':
            temp = (Panel *)panel_userptr(Pau);
            hide_panel(Pau);
            temp->hide = TRUE;
            temp = (Panel *)panel_userptr(Con);
            if(temp->hide == TRUE)
            {	show_panel(Con);
                temp->hide = FALSE;
            }
            stop = 1;
            if(interrupt_begin == 0) interrupt_begin = time(NULL);
            break;
        case 'Q':
        case 'q':
            QuitMenu(Pau,x,y);
            break;
        case KEY_END:
            endwin();
            exit(0);
    }
}

/*
 * 退出菜单
 * 以下操作主要实现以下要求:
 * 1. 创建一个新窗口并覆盖原来的窗口.
 * 2. 然而, 如果玩家点击 "No", 当前窗口消失并使原来的窗口浮现.
 * 3. 如果玩家点击 "Yes", 关闭所有的窗口并结束游戏进程.
 * 4. 否则, 玩家仍可以继续玩游戏.
 */
void QuitMenu(PANEL *Pau, int x, int y)
{
    ITEM **my_items;
    MENU *my_menu;
    WINDOW *my_menu_win;
    Panel *temp;
    int n_quitMenu, i;

    /* Curses 初始化 */
    initscr();
    start_color();
    cbreak();
    noecho();
    keypad(stdscr, TRUE);
    init_pair(1, COLOR_WHITE, COLOR_BLACK);
    init_pair(2, COLOR_RED, COLOR_YELLOW);

    /* 创建菜单和选项 */
    n_quitMenu = ARRAY_SIZE(quitMenu);
    my_items = (ITEM **)calloc(n_quitMenu + 1, sizeof(ITEM *));
    for(i = 0; i < n_quitMenu; i++)
        my_items[i] = new_item(quitMenu[i], quitMenu[i]);
    my_items[n_quitMenu] = (ITEM *)NULL;
    my_menu = new_menu(my_items);

    /* 使窗口位于正中央 */
    menu_opts_off(my_menu, O_SHOWDESC);
    int lent=(x-quit_length) / 2;
    int widt=(y-quit_width) / 2;
    my_menu_win = newwin(8, 35, widt, lent);
    keypad(my_menu_win, TRUE);

    /* 设置主窗口和子窗口 */
    set_menu_win(my_menu, my_menu_win);
    set_menu_sub(my_menu, derwin(my_menu_win,2, 26, 4, 6));
    set_menu_format(my_menu, 1, 3);
    set_menu_mark(my_menu, ">");
    box(my_menu_win, 0, 0);
    wattron(my_menu_win,COLOR_PAIR(2));
    mvwprintw(my_menu_win, 1, 1,"    Are you really want to quit? ");
    wattroff(my_menu_win,COLOR_PAIR(2));
    post_menu(my_menu);
    wrefresh(my_menu_win);
    int check = 0;
    while(check != 1)
    {
        switch(ch)
        {
            /* 每一次 ch = getch()后, 需要给 ch 一个初始值. 否则, 报告错误. */
            case KEY_LEFT:
                menu_driver(my_menu, REQ_PREV_ITEM);
                ch = 0;
                break;
            case KEY_RIGHT:
                menu_driver(my_menu, REQ_NEXT_ITEM);
                ch = 0;
                break;
            case 10:	/* 这是 Enter 键 */
            {
                if(item_name(current_item(my_menu)) == "Yes")
                {
                    ch = 0;
                    wclear(my_menu_win);
                    endwin();
                    delwin(subwin(my_menu_win, 2, 26, 4, 6));
                    delwin(my_menu_win);
                    endwin();
                    exit(0);
                }
                /* 清空退出菜单并返回游戏 */
                if(item_name(current_item(my_menu)) == "No")
                {
                    wclear(my_menu_win);
                    endwin();
                    delwin(subwin(my_menu_win, 2, 26, 4, 6));
                    delwin(my_menu_win);
                    check = 1;
                    ch = 0;
                    break;
                }
                /* 清空退出菜单并重新开始游戏 */
                if(item_name(current_item(my_menu)) == "Replay")
                {
                    check = 1;
                    temp = (Panel *)panel_userptr(Pau);
                    hide_panel(Pau);
                    temp->hide = TRUE;
                    endwin();
                    clear();
                    Initialization();
                    GameInterface();
                    ch = 0;
                    break;
                }
            }
        }
        wrefresh(my_menu_win);
    }
    unpost_menu(my_menu);
    free_menu(my_menu);
    for(i = 0; i < n_quitMenu; ++i)
        free_item(my_items[i]);
}
```
## Collision Problem
我把碰撞问题分成2部分. 第一部分是子弹射中敌人: 对于蜘蛛和蝎子只需要一个子弹就够了, 蜈蚣的单个身节也只需要一个但蘑菇需要4枚子弹才能完全消除. 然而, 射中蜈蚣不同位置获得的奖励分数也不同. 另外, 蜈蚣被击中的部位可能会长出新的头部, 我已把所有我能想到的情况罗列在下方. 如果玩家把蜈蚣全部身节都射中了, 玩家就能成功进入下一关.

<p align="center"><img src="https://github.com/Hephaest/AtariCentipedeGame/blob/master/images/coli.png" width = "400"></p>

第二部分是敌人成功击中玩家, 玩家会失去一次生命当没有多余的生命之后宣告玩家游戏失败. 在被击中后, 如果玩家还有多余的生命, 玩家可以选择重新挑战, 然而, 每一个蘑菇需要4枚子弹才能消除. 所有的敌人将从初始位置重新出现.

您可能会好奇如何实现蜈蚣一分为二的. 其实, 只需要在结构体里定义一个整型 `Clear` 就可以追踪蜈蚣的每个身节. 比如, 玩家射中第 `ith` 个身节, 只需要将此位置 `Centipede[i]` 记录它的状态. 清除, 所有身节的初始清除状态值都是 -1. 在这之后, 遍历所有清除值 `-1` 的身节并使头部的清除值大于或等于 `0` (这意味着头部没有被射中并且它们是蜈蚣的第一个身节). 我不需要关系其他身节清除值的变化, 在移动中只需要让它们跟从头节点变化即可.

<p align="center"><img src="https://github.com/Hephaest/AtariCentipedeGame/blob/master/images/clear.png" width = "400"></p>

```C
/*
 * 碰撞检验
 * 以下操作主要旨在实现以下要求：
 * 1. 一旦子弹射中目标, 程序需要计算分数并确认是否使其消失.
 * 2. 一旦敌人攻击到玩家, 程序判定玩家当此游戏失败并确认玩家是否还有多余的生命.
 */
void CollisionCheck(WINDOW *win,WINDOW *win0,int boundary,PANEL *Pau)
{
    /* 子弹射中蘑菇, 蘑菇只有经受4次子弹攻击才会被销毁 */
    for(int i = 0; i < mushLength; i++) {
        if (bullet_x == mushroom[i].x && bullet_y == mushroom[i].y)
        {
            mushroom[i].mush_record -=1;
            score += 0;// 略过
            mvwprintw(win, 1, 1, "Score: %d", score);
            if (mushroom[i].mush_record == 0) {
                i++;
                while (i < mushLength) {
                    mushroom[i - 1].x = mushroom[i].x;
                    mushroom[i - 1].y = mushroom[i].y;
                    mushroom[i - 1].mush_record = mushroom[i].mush_record;
                    i++;
                }
                mushLength -= 1;
                score += 1;// 如果蘑菇被销毁，获得1分奖励
                mvwprintw(win, 1, 1, "Score: %d", score);
            }
            Fire = 0;
        }
    }
    /* 当蜘蛛碰到蘑菇, 蘑菇被蜘蛛吃掉 */
    for(int i = 0; i < mushLength; i++) {
        if (Spider[1].x == mushroom[i].x && Spider[1].y == mushroom[i].y)
        {
            mushroom[i].mush_record = 0;
            i++;
            while (i < mushLength)
            {
                mushroom[i - 1].x = mushroom[i].x;
                mushroom[i - 1].y = mushroom[i].y;
                mushroom[i - 1].mush_record = mushroom[i].mush_record;
                i++;
            }
            mushLength -= 1;
        }
    }
    /* 一旦蜘蛛或蝎子碰到玩家, 玩家死亡 */
    for(int i = 0; i < 3; i++)
    {
        if((Spider[i].x == master_1_x && Spider[i].y == master_1_y) || (Spider[i].x == master_1_x + 1 && Spider[i].y == master_1_y) || (Spider[i].x==master_1_x+2 && Spider[i].y == master_1_y) || 
           (Spider[i].x == master_2_x && Spider[i].y == master_2_y) || (Spider[i].x == master_2_x + 2 && Spider[i].y == master_2_y) || 
           (Spider[i].x == master_3_x &&  Spider[i].y == master_3_y) || (Spider[i].x == master_3_x + 1 && Spider[i].y == master_3_y) || (Spider[i].x==master_3_x+2 && Spider[i].y == master_3_y))
        {
            end(win0,boundary,Pau);
        }
    }

    for(int i=0;i<2;i++)
    {
        if((Sea_Monster[i].x == master_1_x && Sea_Monster[i].y == master_1_y) || (Sea_Monster[i].x == master_1_x + 1 && Sea_Monster[i].y == master_1_y) || (Sea_Monster[i].x == master_1_x + 2 && Sea_Monster[i].y == master_1_y) || 
           (Sea_Monster[i].x == master_2_x && Sea_Monster[i].y == master_2_y) || (Sea_Monster[i].x == master_2_x + 2 && Sea_Monster[i].y == master_2_y) || 
           (Sea_Mo nster[i].x == master_3_x && Sea_Monster[i].y == master_3_y) || (Sea_Monster[i].x == master_3_x + 1 && Sea_Monster[i].y == master_3_y) || (Sea_Monster[i].x == master_3_x + 2 && Sea_Monster[i].y == master_3_y))
        {
            end(win0,boundary,Pau);
        }
    }


    /* 一旦子弹射中蜘蛛或蝎子, 它们被立即消灭*/
    for(int i = 0; i < 2; i++) {
        if (bullet_x == Sea_Monster[i].x && bullet_y == Sea_Monster[i].y)
        {
            score += 600;// 当此射击获得600分奖励
            mvwprintw(win, 1, 1, "Score: %d", score);
            Sea_Monster[0].x = -1;
            Sea_Monster[1].x = -1;
            sea_appear = 0;
            Fire = 0;
        }
    }

    for(int i = 0; i < 3; i++) {
        if (bullet_x == Spider[i].x && bullet_y == Spider[i].y)
        {
            score += 600;// 当此射击获得600分奖励
            mvwprintw(win, 1, 1, "Score: %d", score);
            Spider[0].x = -1;
            Spider[1].x = -1;
            Spider[2].x = -1;
            spider_appear = 0;
            Fire = 0;
        }
    }
    /* 一旦子弹打中蜈蚣, 程序应对打中情况分类讨论 */
    for(int i = 0; i < Length; i++)
    {
        if(bullet_x == Centipede[i].x && bullet_y == Centipede[i].y)
        {
            /* 如果打中的部位没有任何标记 */
            if(Centipede[i].Clear < 0)
            {
                /* 所有下述情况都有一个共同结果: 打中身节变成蘑菇 */
                mushLength += 1;
                mushroom[mushLength - 1].x = Centipede[i].x;
                mushroom[mushLength - 1].y = Centipede[i].y;
                mushroom[mushLength - 1].mush_record = 4;
                Centipede[i].Clear = i;
                /* 如果打中的身节是尾身节 */
                if(Centipede[i + 1].Clear >= 0 && i + 1 < Length)
                {
                    /* 如果打中的是头部 */
                    if (i == Centipede[i].head)
                    {
                        score += 100;// 当此射击获得100分奖励
                        mvwprintw(win, 1, 1, "Score: %d",score);
                    }
                    else
                    {
                        score += 10;// 当此射击获得10分奖励
                        mvwprintw(win, 1, 1, "Score: %d",score);
                    }
                    Fire = 0;
                    break;
                }
                /* 如果打中的身节不是尾身节 */
                if(Centipede[i + 1].Clear < 0 && i + 1 < Length)
                {
                    // 蜈蚣分裂成两个, 并长出新的头部
                    Centipede[i + 1].head = i + 1;
                    /* 如果打中的是头部 */
                    if (i == Centipede[i].head)
                    {
                        score += 100;// 当此射击获得100分奖励
                        mvwprintw(win, 1, 1, "Score: %d",score);
                    }
                    else
                    {
                        score += 10;// 当此射击获得10分奖励
                        mvwprintw(win, 1, 1, "Score: %d",score);
                    }
                    Fire=0;
                    break;
                }
            }
        }
        /* 一旦蜈蚣击中玩家, 程序应对打中情况分类讨论 */
        if(Centipede[i].Clear<0)
        {
            if((Centipede[i].x == master_1_x && Centipede[i].y == master_1_y) || (Centipede[i].x == master_1_x + 1 && Centipede[i].y == master_1_y) || (Centipede[i].x == master_1_x + 2 && Centipede[i].y == master_1_y) || 
               (Centipede[i].x == master_2_x && Centipede[i].y == master_2_y) || (Centipede[i].x == master_2_x + 2 && Centipede[i].y == master_2_y) || (Centipede[i].x==master_3_x && Centipede[i].y == master_3_y)||(Centipede[i].x == master_3_x + 1 && Centipede[i].y == master_3_y) || (Centipede[i].x == master_3_x + 2 && Centipede[i].y == master_3_y))
            {
                end(win0, boundary, Pau);
                break;
            }
        }
    }
}

/*
 * 此函数将玩家的死亡分类成两种情况.
 * 以下操作主要旨在实现以下要求:
 * 1. 一旦玩家失去生命, 他们有权选择是否重试一遍.
 * 2. 然而, 一旦玩家失去所有生命, 宣告玩家挑战失败.
 */
void end(WINDOW* win, int boundary, PANEL *Pau) {
    int x;
    int y;
    int start_x;
    int start_y;
    int start1_x;
    int start2_x;
    life--;
    if(life > 0)
    {
        getmaxyx(win, y, x);
        start_x = (x - fake_lost) / 2; 
        start1_x = (x - fake_lost_w) / 2;
        start2_x = (x - chance_left) / 2;
        start_y = (y - 3) / 2;
        int Times = 3;
        interrupt_begin = time(NULL);
        while(Times > 0)
        {
            wclear(win);
            mvwprintw(win,start_y, start_x, "You lost!");
            mvwprintw(win,start_y + 1, start1_x, "%d second later you will try again!", Times);
            mvwprintw(win,start_y + 2, start2_x, "Chance left: %d", life);
            wrefresh(win);
            usleep(SECOND);
            Times--;
        }
        interrupt_end = time(NULL);
        wclear(win);
        Fire = 0;
        /* 重启所有角色, 包括新生成的蘑菇 */
        MasterProduce(win);
        Reset_Sea();
        Reset_Spider();
        CentipedeProduce(boundary);
        Reset_Mushroom(win);
    }
    else
    {
        getmaxyx(win, y, x);
        start_x=(x - lost) / 2;
        start_y=(y - 3) / 2;
        wclear(win);
        mvwprintw(win,start_y, start_x, "Game Over! you lost!");
        mvwprintw(win,start_y + 1, start_x, "Your score: %d", score);
        mvwprintw(win,start_y + 2, start_x, "Playtime: %d m %d s", min, sec);
        wrefresh(win);
        while(ch != 'Q'||ch != 'q')
        {
            switch(ch)
            {
                case 'Q':
                case 'q':
                    QuitMenu(Pau,x,y);
                    ch = 0;
                    break;
            }
            mvwprintw(win, start_y, start_x, "Game Over! you lost!");
            mvwprintw(win, start_y + 1, start_x, "Your score: %d", score);
            mvwprintw(win, start_y + 2, start_x, "Playtime: %d m %d s", min, sec);
            wrefresh(win); 
        }
    }
}

/*
 * 该函数用于检验玩家是否消灭了蜈蚣所有的身节.
 * 以下操作主要旨在实现以下要求:
 * 1. 每当玩家成功击中蜈蚣, 程序记录击中的次数.
 * 2. 如果击中次数等于蜈蚣总长, 恭喜玩家进入下一关. 否则, 不发生任何变动.
 */
int success(WINDOW* win,int boundary,PANEL *Pau)
{
    int count=0;
    int x;
    int y;
    int start_x;
    int start_y;
    int start1_x;

    for(int i=0;i<Length;i++)
    {
        if(Centipede[i].Clear>=0) count++;
    }
    if(count == Length)
    {
        Level += 1;
        /* 总共只有5个关卡. */
        if(Level == 6)
        {
            getmaxyx(win, y, x);
            start_x=(x - WIN) / 2;
            start_y=(y - 2) / 2;
            wclear(win);
            mvwprintw(win, start_y, start_x, "You win!");
            mvwprintw(win, start_y + 1, start_x, "Your score: %d", score);
            mvwprintw(win, start_y + 2, start_x, "Playtime: %d m %d s", min, sec);
            wrefresh(win);
            while(ch != 'Q'||ch != 'q')
            {
                switch(ch)
                {
                    case 'Q':
                    case 'q':
                        QuitMenu(Pau,x,y);
                        break;
                }
            }
        }
        else
        {
            getmaxyx(win, y, x);
            start_x = (x - Congrat) / 2;
            start1_x = (x - Congrat_w) / 2;
            start_y = (y - 2) / 2;
            int Times = 3;
            interrupt_begin = time(NULL);
            while(Times > 0)
            {
                wclear(win);
                mvwprintw(win,start_y, start_x,"Congratulations!");
                mvwprintw(win,start_y + 1, start1_x,"%d second later you will go to level %d",Times,Level);
                wrefresh(win);
                usleep(SECOND);
                Times--;
            }
            interrupt_end = time(NULL);
            /*
             * 如果玩家挑战成功, 更改所有角色的颜色并增长蜈蚣长度.
             * 重新设置所有角色的初始位置.
             */
            wclear(win);
            Fire = 0;
            changeColor();
            MasterProduce(win);
            Reset_Sea();
            Reset_Spider();
            CentipedeProduce(boundary);
            wrefresh(win);
        }
    }
    /* 如果只剩下最后一个身节未被击中, 加速蜈蚣运行的速度 */
    else if(count == Length - 1) usleep(SHORT_DELAY);
    else usleep(DELAY);
}
```
## Canvas Refreshment

I debugged many times until I found the ideal steps of canvas refreshment (see below). I fix out the problem which could make spider walk slow by adding new int type variable `delay_spider`, like switch I could turn on or turn off. I do this process when repainting canvas. There is nothing else worth saying, the process is clear in my flow chart.
<p align="center"><img src="https://github.com/Hephaest/AtariCentipedeGame/blob/master/images/game.png"></p>

```C
/*This function is used to create game interface*/
void GameInterface()
{

    WINDOW *my_wins[8];
    PANEL  *my_panels[6];
    Panel panel_datas[6];
    int i = 0, max_y = 0, max_x = 0, win_6_x, win_6_y;
    score = 0; min = 0; sec = 0;
    interrupt = 0; interrupt_begin = 0; interrupt_end = 0;
    begin = 0; over = 0;
    
    /* Curses initialization */
    initscr();
    noecho();//don't display input character
    start_color();
    init_pair(8, COLOR_MAGENTA, COLOR_BLACK);
    init_pair(7, COLOR_CYAN, COLOR_BLACK);
    init_pair(6, COLOR_BLUE, COLOR_BLACK);
    init_pair(5, COLOR_YELLOW, COLOR_BLACK);
    init_pair(4, COLOR_GREEN, COLOR_BLACK);
    init_pair(3, COLOR_RED, COLOR_BLACK);
    curs_set(FALSE);

    int listener;
    listener = pthread_create(& work, NULL, waitForKey, NULL);//Create thread for master
    if (listener != 0) {
        exit(1);
    }

    /* The following operations mainly aim to implement the following requirements:
     * 1. Create 7 windows and ball window will cover the big window.
     * 2. Besides, other windows are used for showing score, pause, continue and quit.
     * 3. Except for big windows, each window need a panel.
     * 4. Only the panel of continue need to hide at beginning.
     */
    getmaxyx(stdscr, max_y, max_x);
    /* Create windows for the panels */
    my_wins[0] = newwin(max_y-score_line, max_x, 0, 0);//big window
    my_wins[1] = newwin(score_line, max_x-quit-level-Pause-Continue-Time, max_y-score_line, 0);//score
    mvwprintw(my_wins[1], 1, 1, "Score: %d", score);
    my_wins[7] = newwin(score_line,Time,max_y-score_line,max_x-quit-level-Pause-Continue-Time);//time
    mvwprintw(my_wins[7], 1, 1, "Time: ");
    my_wins[2] = newwin(score_line, level, max_y-score_line, max_x-quit-level-Pause-Continue);//level
    my_wins[3] = newwin(score_line, Pause, max_y-score_line,max_x-quit-Pause-Continue);//Pause
    mvwprintw(my_wins[3], 1, 1, "\"P\"ause");//Pause label
    my_wins[4] = newwin(score_line, Continue, max_y-score_line, max_x-quit-Continue);//Replay
    mvwprintw(my_wins[4], 1, 1, "\"C\"ontinue"); //Continue label
    my_wins[5] = newwin(score_line, quit, max_y-score_line, max_x-quit);//quit
    mvwprintw(my_wins[5], 1, 1, "\"Q\"uit");//quit label
    my_wins[6] = newwin(max_y-score_line-2, max_x-2, 1, 1);// ball
    getmaxyx(my_wins[6], win_6_y, win_6_x);
    keypad(stdscr, TRUE);

    for(i = 0; i < 6; i++) box(my_wins[i], 0, 0);
    for(i = 0; i < 6; i++) my_panels[i] = new_panel(my_wins[i]);
    my_panels[6] = new_panel(my_wins[7]);
    for(i = 0; i < 6; i++)
    {
        panel_datas[i].hide = FALSE;
        set_panel_userptr(my_panels[i], &panel_datas[i]);
    }
    panel_datas[6].hide = FALSE;
    set_panel_userptr(my_panels[6], &panel_datas[7]);
    panel_datas[4].hide = TRUE;
    hide_panel(my_panels[4]);

    /* Show it on the screen */
    doupdate();
    /*Produce mushrooms, a master and a centipede at first*/
    MushroomProduce(win_6_y,win_6_x-1);
    MasterProduce(my_wins[6]);
    CentipedeProduce(win_6_x);
    /*If player do not lose all lives, the game will continue. Else, quit*/
    begin = time(NULL);
    while(life > 0)
    {
        wclear(my_wins[6]);
        for(i = 0; i < mushLength; i++)
        {
            wattron(my_wins[6],COLOR_PAIR(mushroom_color));
            mvwprintw(my_wins[6],mushroom[i].y,mushroom[i].x,"&");
            wattroff(my_wins[6],COLOR_PAIR(mushroom_color));
        }
        wattron(my_wins[6],COLOR_PAIR(centipede_color));
        for(i = 0; i < Length; i++)
        {
            if(Centipede[i].Clear<0)
            {

                if(i == Centipede[i].head) mvwprintw(my_wins[6],Centipede[i].y,Centipede[i].x,"O");
                else mvwprintw(my_wins[6],Centipede[i].y,Centipede[i].x,"*");
            }
        }
        wattroff(my_wins[6],COLOR_PAIR(centipede_color));
        if(sea_appear)
        {
            wattron(my_wins[6],COLOR_PAIR(sea_color));
            mvwprintw(my_wins[6],Sea_Monster[0].y, Sea_Monster[0].x, "$");
            mvwprintw(my_wins[6],Sea_Monster[1].y, Sea_Monster[1].x, "_");
            wattroff(my_wins[6],COLOR_PAIR(sea_color));
        }
        if(Spider[1].x != -1 & spider_appear)
        {
                wattron(my_wins[6],COLOR_PAIR(spider_color));
                mvwprintw(my_wins[6],Spider[0].y, Spider[0].x, "^");
                mvwprintw(my_wins[6],Spider[1].y, Spider[1].x, "W");
                mvwprintw(my_wins[6],Spider[2].y, Spider[2].x, "^");
                wattroff(my_wins[6],COLOR_PAIR(spider_color));
        }
        else SpiderProduce(win_6_y,win_6_x);

        mvwprintw(my_wins[2], 1, 1, "Level: %d",Level);
        getInt(my_panels[4],my_panels[3],win_6_x,win_6_y,my_wins[6]);
        if(stop<1)
        {
            wattron(my_wins[6],COLOR_PAIR(master_color));
            getOrder(my_wins[6],win_6_x,win_6_y);
            wattroff(my_wins[6],COLOR_PAIR(master_color));
            wattron(my_wins[6],COLOR_PAIR(bullet_color));
            if(Fire)
            {
                bullet_y -= 1;
                mvwprintw(my_wins[6],bullet_y,bullet_x,"|");
            }
            else
            {
                bullet_y = master_1_y;
                bullet_x = master_1_x+1;
            }
            wattroff(my_wins[6],COLOR_PAIR(bullet_color));
            wrefresh(my_wins[2]);
            wrefresh(my_wins[1]);
            wrefresh(my_wins[6]);
            CollisionCheck(my_wins[1],my_wins[6],win_6_x,my_panels[3]);
            success(my_wins[6],win_6_x,my_panels[3]);
            if(sea_appear)
            {
                if(Sea_Monster[0].x != -1) sea_monsterMove(win_6_x);
                else Sea_MonsterProduce(win_6_y,win_6_x);
            }

            if(spider_appear && delay_spider == 1) SpiderMove(win_6_x,win_6_y);

            CentipedeMove(win_6_x - 1, win_6_y - 1);
            delay_spider *= -1;
            //timer
            over = time(NULL);
            double seconds = difftime(over,begin);
            if(difftime(interrupt_end,interrupt_begin)>0)
            {
                interrupt += difftime(interrupt_end,interrupt_begin);
                interrupt_begin = 0;
                interrupt_end = 0;
            }
            seconds -= interrupt;
            if(seconds >= 60)
            {
                min = ((int)seconds) / 60;
                sec = ((int)seconds) - min*60;
            }
            else sec=(int)seconds;
            wclear(my_wins[7]);
            box(my_wins[7], 0, 0);
            mvwprintw(my_wins[7], 1, 1, "Time: %d m %d s",min,sec);
            wrefresh(my_wins[7]);
        }
        update_panels();
        doupdate();
    };
    endwin();
}
```

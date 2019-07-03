# Atari Centipede Game By C
A C language code to imitate the Atari Centipede Game.<br/>
Hope you like my version of the centipede game and have fun. :)<br/>
It's recommended that you read this readme.md firstly then try to run the program on your Linux device.<br/>
You can see the original game in here: https://www.youtube.com/watch?v=dxoK8hosHjA&t=1s!<br/>

# Introduction
**ATTENTIONS**: This game requires `ncurses` library, which only could be used in Unix OS. Before I start, I suggest you to install this library by the following command in Debian/Ubuntu Linux:
```C
sudo apt−get install libncurses5−dev libncursesw5−dev
//libncurses5−dev : Developer's libraries for ncurses
//libncursesw5−dev : Developer's libraries for ncursesw
```
## Restatement
Based on the arcade original, I rewrite the rules of my own game. In order to make my game more stable, clear and smooth, I make some essential assumptions.
### Game Regulations
**For master**<br>
I call the player as ’Master’, Master could turn left, right, up and down by direction key. In addition, if player wants to fire a bullet, press space key can easily make it work. Just remember master only has 4 lives, be careful!<br>

**For mushroom**<br>
It is the main obstacle for players to move and fire. Mushrooms can be destroyed after taking four shots or eating by spider.<br>

**For spider**<br>
It is one kind of the headstrong guy in this game. It could either move up or down or move around. The moving tracks are similar to ’W’ and ’I’. In addition, it is also some people tern a ’frienemy’- If it hits mushroom, mushroom will be destroyed. Conversely, master will die once spider hits him. For the sake of security, I strongly advise players to avoid it unless they could shoot it.<br>

**For sea monster**<br>
In Atari Centipede, it has scorpion. But here, I have sea monster because I think it is cool. It is another headstrong guy in this game. Compared to spider, it is totally a bad guy who is always loafing around horizontally and doing nothing but hitting master and making it die.<br>

**For centipede**<br>
It is the major target of players. If nothing happens, centipede just walks line by line. Every time player shoots the centipede, the shot segment becomes a mush-room. Then if the shot segment is no in the tail or the body centipede only leaves, the centipede will split into two, gain a new head and both descend towards the player but ascend towards the player if it is in the floor. Besides, if centipede hits the wall or mushroom, it will try to turn back and descend towards the player if no other mushroom and centipede in where it will descend. Keep yourself away from centipede in case of hitting, which gonna takes you life!<br>

To make the above game regulations easier to understand, I created a table as follows:

|       Hit       |                       Master                       |                        bullets                         |                           Mushroom                           |       Centipede        |
| :-------------: | :------------------------------------------------: | :----------------------------------------------------: | :----------------------------------------------------------: | :--------------------: |
|   **Spider**    |             Master die, lose one life              |               Spider die, win 600 points               |      Mushroom die, the number of mushroom decreases one      |         Ignore         |
| **Sea monster** |             Master die, lose one life              |            Sea monster die, win 600 points             |                            Ignore                            |         Ignore         |
|  **Centipede**  |             Master die, lose one life              | win 100 points if hitting its head, else win 10 points | turn back and descend If no other mushrooms under centipede, else if its head is between mushroom and wall, just descend.  Centipede turns back in other cases. | One of them turns back |
|  **Mushroom**   | Mushroom die, the number of mushroom decreases one |                    4 hits = 1 point                    |                            Ignore                            |  No kind of this case  |

Further, I set different levels for this game. The initial lengths of centipede in different levels are the same but this doesn’t mean easy. The more mushrooms there are, the harder it is for the player to shoot the centipede. Needless to say spider and sea monster.

### Assumption
While I ran my own game, I met some unexpected circumstances. Therefore, I list some necessary assumptions:
- At the beginning of each level, If mushroom position coincides with the ini-tial position of the centipede, then player could believe that these mush-rooms have been ‘eaten’ by centipede.
- The spider only appears in the lower part of the game window. Conversely, the sea monster only appears in the upper part of the game window.
- The player’s scope of activity should not exceed the boundary of the win-dow. In addition, Master couldn’t hit mushroom but bullet could.
- The shot segments become mushrooms, these mushrooms will always be there whichever level is being played or whatever win or lose unless player use 4 bullets to destroyed them.

# Game Flow Chart
<p align=center>
<img src="https://github.com/Hephaest/AtariCentipedeGame/blob/master/images/flow.png" width="250"/>
</p>
This figure shows all the processes of the game in detail, including the collision re-sults, score updates and characters movements, etc. In this process, each time computer needs to get order from I/O and determine the collision judgment whether is satisfied or not, and then, refresh the canvas and update the scores.<br>
If you’re unfamiliar with this process, you can see my game screen recorder in YouTube:<br>
https://www.youtube.com/watch?v=iQRfnJUAYQs<br>
Or watch the original flow chart:<br>
https://www.processon.com/view/link/5b040ea4e4b05f5d6b641243

# Function Implementations
## Start Menu
Firstly, I want to create a home page menu to give a short introduction about the game rule. This roll-up menu have 2 choices: play or not.
```C
/* These choices are used in the introduction menu*/
char *startMenu[] = {
        "Start", "Exit",
};

/* The main content of the introduction menu*/
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

/*These choices are used in the quit menu*/
char *quitMenu[] = {
        "No",
        "Yes",
        "Replay",
};
```
After that, I need to create a window to show this menu. Because I couldn’t find a way to make my text scroll, so I just put them all into menu but only give my text scroll attribution.
```C
/*This function is the Introduction Menu User Pointer Usage*/
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

    /* Create introduction menu items.
     * The following operations mainly aim to implement the following requirement:
     * 1. Readme only shows words and could scroll.
     * 2. Highlight all choices of startMenu.
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
    /*Set the last one is NULL*/
    start_items[n_readme + n_startMenu] = (ITEM *)NULL;
    /* Crate menu */
    menu = new_menu((ITEM **)start_items);

    /* This color will be used in startMenu's choices*/
    set_menu_fore(menu, COLOR_PAIR(2));
    set_menu_back(menu, COLOR_PAIR(1));
    set_menu_grey(menu, COLOR_PAIR(1));
    menu_opts_off(menu, O_SHOWDESC);

    /* Create the window and make it at center*/
    int max_y, max_x;
    getmaxyx(stdscr, max_y, max_x);
    int x_position = (max_x-box_length)/2;
    int y_position = (max_y-box_width)/2;
    start_menu = newwin(box_width, box_length, y_position, x_position);
    keypad(start_menu, TRUE);//To listen keyboard

    /* Set main window and sub window */
    set_menu_win(menu, start_menu);
    set_menu_sub(menu, derwin(start_menu, box_width-1, box_length-2, 1, 1));
    set_menu_format(menu, 18, 1);// Set 18 lines inside the box and one choices each line

    /*Create borders around the windows*/
    box(start_menu, 0, 0);
    refresh();

    /* Post the menu */
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

    /* Unpost and free all the memory taken up */
    unpost_menu(menu);
    for(int i = 0; i < n_startMenu + n_readme; ++i)
        free_item(start_items[i]);
    free_menu(menu);
    /*Quit this window and wait for another*/
    endwin();
}
```
The menu items have been divided into two parts: the 1st part is text(I don’t want computer do more thing for it). But in the second part, I set user pointers to allow player make a choice. In order to remind player to choose, I highlight the background of choices. 
```C
/*Filter choices*/
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
The running result of above codes is shown as follows.

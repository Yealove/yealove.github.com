---
date: 2015-06-23 00:18
tags:
- Java
- Tool
title: java显示当前鼠标位置
---

```java
import javax.swing.*;
import java.awt.*;

public class MousePosition extends JFrame {
    public static void main(String[] args) {
        JFrame fm = new JFrame("鼠标坐标");

        //获取屏幕大小
        Dimension screensize = Toolkit.getDefaultToolkit().getScreenSize();
        int width = (int) screensize.getWidth();
        //int height = (int)screensize.getHeight();

        fm.setLocation(width - 250, 40);
        fm.setSize(200, 65);
        fm.setResizable(false);

        // 去掉窗口的装饰
        fm.setUndecorated(true);
        //采用指定的窗口装饰风格
        fm.getRootPane().setWindowDecorationStyle(JRootPane.INFORMATION_DIALOG);

        fm.setVisible(true);
        fm.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JPanel panel = new JPanel();

        JLabel lab = new JLabel();
        panel.add(lab);

        fm.add(panel);

        while (true) {
            //获取鼠标位置
            Point point = MouseInfo.getPointerInfo().getLocation();
            lab.setText("x:" + point.x + ", y:" + point.y);
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                fm.setResizable(true);
                lab.setText(e.getMessage());
            }
        }
    }
}
```
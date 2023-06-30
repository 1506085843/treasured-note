[TOC]

# 前言
本文使用的是 jdk8 的 javafx 运行实现的图片缩放操作效果。
# 方式一
通过改变 ImageView 的 FitHeight、FitWidth 来改变及调整长宽来缩放，你可以参考这篇文章[JavaFX图片浏览并实现缩放](https://blog.csdn.net/m0_37649480/article/details/125941031)

fxml文件里ImageView如下：
```xml
<ImageView fx:id="image" fitHeight="600.0" fitWidth="600.0" pickOnBounds="true" preserveRatio="true" />
```
java代码如下：
```java
     static double size = 1;
     static double count = 1.0;
    
     //鼠标滚轮控制图片缩放
     image.setOnScroll(event -> {
            // 缩放具体逻辑
            if (event.getDeltaY() > 0) {
                // 这里是向上滚动滚轮（即放大图片）
                count = count + 1.0 / 10;
                size = 1.0 / 200 * (count - 1) * (count - 1) * (count - 1) + 1;
                image.setFitWidth(image.getFitWidth() * size);
                image.setFitHeight(image.getFitHeight() * size);
                count++;
            } else {
                // 这里是乡下滚动滚轮（即缩小图片）
                count = count - 1.0 / 10;
                double y = 1.0 / 200 * (count - 1) * (count - 1) * (count - 1) + 1;
                size = y < 0 ? size : y;
                image.setFitWidth(image.getFitWidth() / size);
                image.setFitHeight(image.getFitHeight() / size);
                count--;
            }
     });
```

# 方式二
上面的方式确实能够实现鼠标滚轮进行图片缩放，但它缩放时 ImageView 所在的整个 stage 页面也会缩放。
那怎么让图片在一个框里缩放呢，也就是让图片只在指定框的区域显示并缩放，你可以使用 ImageView 里的 setViewport 方法并在里面使用 Rectangle2D 设置图形的位置和宽高，
你可以参考这篇文章：[Zooming an Image in ImageView (javafx)](https://stackoverflow.com/questions/48687994/zooming-an-image-in-imageview-javafx)

## 1.带有滚动条的缩放

### （1）代码

你可以直接复制如下代码，运行便可看到效果（使用jdk8）

```java

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import javafx.application.Application;
import javafx.geometry.Orientation;
import javafx.geometry.Pos;
import javafx.geometry.Rectangle2D;
import javafx.stage.FileChooser;
import javafx.stage.Stage;
import javafx.stage.FileChooser.ExtensionFilter;
import javafx.scene.Cursor;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.control.Label;
import javafx.scene.control.Slider;
import javafx.scene.control.TextField;
import javafx.scene.image.Image;
import javafx.scene.image.ImageView;
import javafx.scene.layout.BorderPane;
import javafx.scene.layout.GridPane;
import javafx.scene.layout.HBox;
import javafx.scene.layout.VBox;

public class MainServer extends Application {
    static double initx;
    static double inity;
    static int height;
    static int width;
    public static String path;
    static Scene initialScene, View;
    static double offSetX, offSetY, zoomlvl;

    @Override
    public void start(Stage s) {
        s.setResizable(false);
        GridPane grid = new GridPane();
        grid.setHgap(20);
        grid.setVgap(20);
        grid.setAlignment(Pos.CENTER);

        Label hint = new Label("你选择的图片：");
        TextField URL = new TextField();
        URL.setEditable(false);
        URL.setPrefWidth(350);

        Button browse = new Button("选择图片");
        //现在只能选择jpg、png格式的图片
        FileChooser fc = new FileChooser();
        ExtensionFilter png = new ExtensionFilter("png", "*.png");
        ExtensionFilter jpg = new ExtensionFilter("jpg", "*.jpg");
        fc.getExtensionFilters().addAll(jpg,png);
        //点击选择图片按钮的事件
        browse.setOnAction(e -> {
            URL.setText(fc.showOpenDialog(s).getAbsolutePath());
        });

        Button open = new Button("打开");
        //点击打开按钮的事件
        open.setOnAction(e -> {
            //获取图片路径到
            path = URL.getText();
            //初始化显示图片
            initView();
            s.setScene(View);
        });

        grid.add(hint, 0, 0);
        grid.add(URL, 1, 0);
        grid.add(browse, 2, 0);
        grid.add(open, 2, 1);

        initialScene = new Scene(grid, 600, 100);
        s.setScene(initialScene);
        s.show();
    }

    public static void initView() {
        VBox root = new VBox(20);
        root.setAlignment(Pos.CENTER);

        Label title = new Label(path.substring(path.lastIndexOf("\\") + 1));
        //加载图片
        Image source = null;
        try {
            source = new Image(new FileInputStream(path));
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }

        ImageView image = new ImageView(source);
        //获取图片长宽比
        double ratio = source.getWidth() / source.getHeight();
        //设置长宽
        if (500 / ratio < 500) {
            width = 500;
            height = (int) (500 / ratio);
        } else if (500 * ratio < 500) {
            height = 500;
            width = (int) (500 * ratio);
        } else {
            height = 500;
            width = 500;
        }
        //设置图片长宽
        image.setPreserveRatio(false);
        image.setFitWidth(width);
        image.setFitHeight(height);
        height = (int) source.getHeight();
        width = (int) source.getWidth();
        System.out.println("height = " + height + "\nwidth = " + width);

        //设置底部缩放滑动条，缩放程度限制为1到4
        HBox zoom = new HBox(10);
        zoom.setAlignment(Pos.CENTER);
        Slider zoomLvl = new Slider();
        zoomLvl.setMax(4);
        zoomLvl.setMin(1);
        zoomLvl.setMaxWidth(200);
        zoomLvl.setMinWidth(200);
        Label hint = new Label("缩放程度");
        Label value = new Label("1.0");
        zoom.getChildren().addAll(hint, zoomLvl, value);

        offSetX = width / 2;
        offSetY = height / 2;

        //设置顶部水平拖动滑动条
        Slider Hscroll = new Slider();
        Hscroll.setMin(0);
        Hscroll.setMax(width);
        Hscroll.setMaxWidth(image.getFitWidth());
        Hscroll.setMinWidth(image.getFitWidth());
        Hscroll.setTranslateY(-20);
        //设置右侧垂直拖动滑动条
        Slider Vscroll = new Slider();
        Vscroll.setMin(0);
        Vscroll.setMax(height);
        Vscroll.setMaxHeight(image.getFitHeight());
        Vscroll.setMinHeight(image.getFitHeight());
        Vscroll.setOrientation(Orientation.VERTICAL);
        Vscroll.setTranslateX(-20);

        //将三个滑动条和图片放到 imageView 布局上
        BorderPane imageView = new BorderPane();
        BorderPane.setAlignment(Hscroll, Pos.CENTER);
        BorderPane.setAlignment(Vscroll, Pos.CENTER_LEFT);
        imageView.setCenter(image);
        imageView.setTop(Hscroll);
        imageView.setRight(Vscroll);
        //设置鼠标手张开的图标
        imageView.setCursor(Cursor.OPEN_HAND);
        //顶部滑动条变化时的出发事件
        Hscroll.valueProperty().addListener(e -> {
            offSetX = Hscroll.getValue();
            zoomlvl = zoomLvl.getValue();
            double newValue = (double) ((int) (zoomlvl * 10)) / 10;
            value.setText(newValue + "");
            if (offSetX < (width / newValue) / 2) {
                offSetX = (width / newValue) / 2;
            }
            if (offSetX > width - ((width / newValue) / 2)) {
                offSetX = width - ((width / newValue) / 2);
            }
            image.setViewport(new Rectangle2D(offSetX - ((width / newValue) / 2), offSetY - ((height / newValue) / 2), width / newValue, height / newValue));
        });
        //右侧滑动条变化时的出发事件
        Vscroll.valueProperty().addListener(e -> {
            offSetY = height - Vscroll.getValue();
            zoomlvl = zoomLvl.getValue();
            double newValue = (double) ((int) (zoomlvl * 10)) / 10;
            value.setText(newValue + "");
            if (offSetY < (height / newValue) / 2) {
                offSetY = (height / newValue) / 2;
            }
            if (offSetY > height - ((height / newValue) / 2)) {
                offSetY = height - ((height / newValue) / 2);
            }
            image.setViewport(new Rectangle2D(offSetX - ((width / newValue) / 2), offSetY - ((height / newValue) / 2), width / newValue, height / newValue));
        });
        //底部滑动条变化时的出发事件
        zoomLvl.valueProperty().addListener(e -> {
            zoomlvl = zoomLvl.getValue();
            double newValue = (double) ((int) (zoomlvl * 10)) / 10;
            value.setText(newValue + "");
            if (offSetX < (width / newValue) / 2) {
                offSetX = (width / newValue) / 2;
            }
            if (offSetX > width - ((width / newValue) / 2)) {
                offSetX = width - ((width / newValue) / 2);
            }
            if (offSetY < (height / newValue) / 2) {
                offSetY = (height / newValue) / 2;
            }
            if (offSetY > height - ((height / newValue) / 2)) {
                offSetY = height - ((height / newValue) / 2);
            }
            Hscroll.setValue(offSetX);
            Vscroll.setValue(height - offSetY);
            image.setViewport(new Rectangle2D(offSetX - ((width / newValue) / 2), offSetY - ((height / newValue) / 2), width / newValue, height / newValue));
        });
        //鼠标在图片上按压的事件
        image.setOnMousePressed(e -> {
            initx = e.getSceneX();
            inity = e.getSceneY();
            imageView.setCursor(Cursor.CLOSED_HAND);
        });
        //鼠标在图片上松开的事件
        image.setOnMouseReleased(e -> {
            imageView.setCursor(Cursor.OPEN_HAND);
        });
        //鼠标在图片上拖动的事件
        image.setOnMouseDragged(e -> {
            Hscroll.setValue(Hscroll.getValue() + (initx - e.getSceneX()));
            Vscroll.setValue(Vscroll.getValue() - (inity - e.getSceneY()));
            initx = e.getSceneX();
            inity = e.getSceneY();
        });
        //鼠标在图片上滚轮滚动控制缩放的事件。缩放程度限制在1到4，每次滚动缩放0.3
        image.setOnScroll(event -> {
            zoomlvl = zoomLvl.getValue();
            if (event.getDeltaY() > 0) {
                if (zoomlvl < 4) {
                    zoomLvl.setValue(zoomlvl + 0.3);
                }
            } else {
                if (zoomlvl > 1) {
                    zoomLvl.setValue(zoomlvl - 0.3);
                }
            }
        });
        root.getChildren().addAll(title, imageView, zoom);
        //设置新窗口的大小
        View = new Scene(root, (image.getFitWidth()) + 90, (image.getFitHeight()) + 170);
    }

    public static void main(String[] args) {
        launch(args);
    }
}

```

### （2）效果

![请添加图片描述](https://img-blog.csdnimg.cn/663527dfa2b24fadb4a45bd944bd506b.jpeg)

![请添加图片描述](https://img-blog.csdnimg.cn/a441a06a4d954525a3506a8161dd0e77.jpeg)

## 2.fxml 布局+java代码
上面的示例是纯使用java来布局并控制逻辑的，并且还带有滑动条。下面调整下使用 fxml  进行布局，用java代码控制逻辑，并隐藏所有滑动条。

你可以参考我下面的写法，如你想运行或看完整的项目你可以看我: [github的XTool项目地址](https://github.com/1506085843/XTool)

### (1) fxml 布局文件

```xml
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.scene.control.*?>
<?import javafx.scene.layout.*?>
<?import javafx.geometry.*?>
<?import com.jfoenix.controls.*?>
<?import javafx.scene.control.*?>
<?import javafx.scene.layout.*?>
<?import javafx.scene.image.ImageView?>

<StackPane fx:id="root" xmlns="http://javafx.com/javafx/8" xmlns:fx="http://javafx.com/fxml/1">
    <children>
        <VBox  maxHeight="1050" maxWidth="1200" spacing="30" >
            <children>
                <HBox spacing="20" alignment="BASELINE_LEFT" style="-fx-padding:130 0 0 0 ">
                    <children>
                        <JFXButton fx:id="fileButton" layoutX="55.0"  style="-fx-background-color: #5cb85c;" text="打开图片" textFill="WHITE"  />
                        <JFXButton fx:id="identify" layoutX="170.0" layoutY="204.0" style="-fx-background-color: #5cb85c;" text="识别" textFill="WHITE" />
                        <JFXButton fx:id="copy" layoutX="199.0" layoutY="204.0" style="-fx-background-color: #5cb85c;" text="复制结果" textFill="WHITE" />
                        <JFXButton fx:id="clear" layoutX="219.0" layoutY="204.0" style="-fx-background-color: #c9302c;" text=" 清空 " textFill="WHITE" />
                    </children>
                </HBox>

                <HBox spacing="20"  maxHeight="550" prefHeight="550" >
                    <children>

                        <SplitPane layoutX="55.0" layoutY="14.0" prefHeight="550.0" prefWidth="700.0" maxHeight="550" style="-fx-padding:10 10 -400 50;-fx-font: 15 arial;" >
                            <VBox  maxHeight="550"  prefHeight="550" style="-fx-padding:0 10 0 40">
                                <children>
                                    <!-- 使用  visible="false" 隐藏顶部滚动条-->
                                    <Slider fx:id="Hscroll"  min="0" translateY="-20" maxHeight="0" maxWidth="0" visible="false"></Slider>
                                    <ImageView fx:id="image"  pickOnBounds="true" preserveRatio="false" />
                                    <!-- 使用  visible="false" 隐藏底部滚动条 使用 maxWidth="0" 让宽度设为0也为了不显示-->
                                    <Slider fx:id="zoomLvl" maxWidth="0"  max="6" min="1" visible="false"></Slider>
                                    <!-- 使用  visible="false" 隐藏底部滚动条的名称 使用 maxHeight、maxWidth 让长宽为0也为了不显示-->
                                    <Label fx:id="hint"  maxHeight="0" maxWidth="0" visible="false">缩放程度</Label>
                                    <!-- 使用  visible="false" 隐藏右侧滚动条 使用 maxHeight、maxWidth 让长宽为0也为了不显示-->
                                    <Slider fx:id="Vscroll" min="0" translateX="-20" orientation="VERTICAL"  maxHeight="0" maxWidth="0" visible="false"></Slider>
                                    <!-- 使用  visible="false" 隐藏底部滚动条的值 使用 maxHeight、maxWidth 让长宽为0也为了不显示-->
                                    <Label fx:id="value"  maxHeight="0" maxWidth="0" visible="false">1.0</Label>
                                </children>
                            </VBox>
                        </SplitPane>

                        <TextArea fx:id="resultArea" layoutX="55.0" layoutY="238.0" prefHeight="550.0" prefWidth="700.0" promptText="识别结果（仅支持识别中英文）" style="-fx-padding: 5 5 5 5;-fx-font: 15 arial;" >
                            <opaqueInsets>
                                <Insets />
                            </opaqueInsets>
                        </TextArea>
                    </children>
                </HBox>

            </children>
        </VBox>
    </children>
</StackPane>


```

### (2) java 代码

```java
import com.jfoenix.controls.JFXButton;
import io.datafx.controller.ViewController;
import javafx.fxml.FXML;
import javafx.geometry.Pos;
import javafx.geometry.Rectangle2D;
import javafx.scene.Cursor;
import javafx.scene.control.Label;
import javafx.scene.control.Slider;
import javafx.scene.control.TextArea;
import javafx.scene.image.Image;
import javafx.scene.image.ImageView;
import javafx.scene.input.Clipboard;
import javafx.scene.input.ClipboardContent;
import javafx.scene.layout.BorderPane;
import javafx.stage.FileChooser;
import javafx.stage.Stage;
import net.sourceforge.tess4j.Tesseract;
import net.sourceforge.tess4j.TesseractException;
import javax.annotation.PostConstruct;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;

@ViewController(value = "/fxml/ui/Ocr.fxml", title = "OCR图片识字")
public class OcrController {
    @FXML
    private JFXButton fileButton;
    @FXML
    private JFXButton identify;
    @FXML
    private JFXButton copy;
    @FXML
    private JFXButton clear;
    @FXML
    private TextArea borderArea;
    @FXML
    private ImageView image;
    @FXML
    private TextArea resultArea;
    @FXML
    private Slider Hscroll;
    @FXML
    private Label hint;
    @FXML
    private Slider zoomLvl;
    @FXML
    private Label value;
    @FXML
    private Slider Vscroll;

    private static double size = 1;
    private static double count = 1.0;
    private String imagePath = "";//图像路径
    private static int width = 0;
    static int height = 0;
    static double ratio = 0;
    private Image source = null;
    private static double offSetX, offSetY, zoomlvl;
    private static double initx;
    private static double inity;

    @PostConstruct
    public void init() {
        //选择文件按钮
        fileButton.setOnAction(action -> {
            FileChooser fileChooser = new FileChooser();
            fileChooser.setTitle("选择图像文件");
            File file = fileChooser.showOpenDialog(new Stage());
            if (file != null) {
                //获取图像路径
                imagePath = file.getAbsolutePath();
                System.out.println(imagePath);
                FileInputStream input = null;
                try {
                    input = new FileInputStream(imagePath);
                } catch (FileNotFoundException e) {
                    e.printStackTrace();
                }
                //获取图像为Image对象
                source = new Image(input);
                //获取长宽比
                getWidthHeight();
                //设置图像的长宽和设置到ImageView对象
                image.setImage(source);
                image.setPreserveRatio(false);
                image.setFitWidth(width);
                image.setFitHeight(height);
                height = (int) source.getHeight();
                width = (int) source.getWidth();

                offSetX = width / 2;
                offSetY = height / 2;
                //设置顶部滚动条
                Hscroll.setMax(width);
                Hscroll.setMaxWidth(image.getFitWidth());
                Hscroll.setMinWidth(image.getFitWidth());
                //设置右侧滚动条
                Vscroll.setMax(height);
                Vscroll.setMaxHeight(image.getFitHeight());
                Vscroll.setMinHeight(image.getFitHeight());
                BorderPane.setAlignment(Hscroll, Pos.CENTER);
                BorderPane.setAlignment(Vscroll, Pos.CENTER_LEFT);
                //顶部滚动条监听
                Hscroll.valueProperty().addListener(e -> {
                    offSetX = Hscroll.getValue();
                    zoomlvl = zoomLvl.getValue();
                    double newValue = (double) ((int) (zoomlvl * 10)) / 10;
                    value.setText(newValue + "");
                    if (offSetX < (width / newValue) / 2) {
                        offSetX = (width / newValue) / 2;
                    }
                    if (offSetX > width - ((width / newValue) / 2)) {
                        offSetX = width - ((width / newValue) / 2);
                    }

                    image.setViewport(new Rectangle2D(offSetX - ((width / newValue) / 2), offSetY - ((height / newValue) / 2), width / newValue, height / newValue));
                });
                //右侧滚动条监听
                Vscroll.valueProperty().addListener(e -> {
                    offSetY = height - Vscroll.getValue();
                    zoomlvl = zoomLvl.getValue();
                    double newValue = (double) ((int) (zoomlvl * 10)) / 10;
                    value.setText(newValue + "");
                    if (offSetY < (height / newValue) / 2) {
                        offSetY = (height / newValue) / 2;
                    }
                    if (offSetY > height - ((height / newValue) / 2)) {
                        offSetY = height - ((height / newValue) / 2);
                    }

                    image.setViewport(new Rectangle2D(offSetX - ((width / newValue) / 2), offSetY - ((height / newValue) / 2), width / newValue, height / newValue));
                });
                //底部缩放滚动条监听
                zoomLvl.valueProperty().addListener(e -> {
                    zoomlvl = zoomLvl.getValue();
                    double newValue = (double) ((int) (zoomlvl * 10)) / 10;
                    value.setText(newValue + "");
                    if (offSetX < (width / newValue) / 2) {
                        offSetX = (width / newValue) / 2;
                    }
                    if (offSetX > width - ((width / newValue) / 2)) {
                        offSetX = width - ((width / newValue) / 2);
                    }
                    if (offSetY < (height / newValue) / 2) {
                        offSetY = (height / newValue) / 2;
                    }
                    if (offSetY > height - ((height / newValue) / 2)) {
                        offSetY = height - ((height / newValue) / 2);
                    }
                    Hscroll.setValue(offSetX);
                    Vscroll.setValue(height - offSetY);
                    image.setViewport(new Rectangle2D(offSetX - ((width / newValue) / 2), offSetY - ((height / newValue) / 2), width / newValue, height / newValue));
                });
                //鼠标在图片上按压的事件
                image.setOnMousePressed(e -> {
                    initx = e.getSceneX();
                    inity = e.getSceneY();
                    image.setCursor(Cursor.CLOSED_HAND);
                });
                //鼠标在图片上松开的事件
                image.setOnMouseReleased(e -> {
                    image.setCursor(Cursor.OPEN_HAND);
                });
                //鼠标在图片上拖动的事件
                image.setOnMouseDragged(e -> {
                    Hscroll.setValue(Hscroll.getValue() + (initx - e.getSceneX()));
                    Vscroll.setValue(Vscroll.getValue() - (inity - e.getSceneY()));
                    initx = e.getSceneX();
                    inity = e.getSceneY();
                });
            }
        });

        //鼠标图片缩放控制
        image.setOnScroll(event -> {
            zoomlvl = zoomLvl.getValue();
            if (event.getDeltaY() > 0) {
                if (zoomlvl < 6) {
                    zoomLvl.setValue(zoomlvl + 0.3);
                }
            } else {
                if (zoomlvl > 1) {
                    zoomLvl.setValue(zoomlvl - 0.3);
                }
            }
        });

        //识别按钮
        identify.setOnAction(action -> {
            if (imagePath.length() == 0) {
                return;
            }
            long start = System.currentTimeMillis();
            //加载要识别的图片
            File image = new File(imagePath);
            //设置配置文件夹微视、识别语言、识别模式
            Tesseract tesseract = new Tesseract();
            tesseract.setDatapath("src/main/resources/tessdata");
            tesseract.setLanguage("chi_sim");
            tesseract.setPageSegMode(1);
            //设置引擎模式
            tesseract.setOcrEngineMode(1);
            //开始识别图片中的文字
            String result = null;
            try {
                result = tesseract.doOCR(image);
            } catch (TesseractException e) {
                e.printStackTrace();
            }
            long time = System.currentTimeMillis() - start;
            System.out.println("识别结束,耗时：" + time + " 毫秒，识别结果如下：");
            System.out.println();
            System.out.println(result);
            resultArea.setText(result);
        });

        //复制按钮事件
        copy.setOnAction(action -> {
            String str = resultArea.getText();
            Clipboard clipboard = Clipboard.getSystemClipboard();
            ClipboardContent content = new ClipboardContent();
            content.putString(str);
            clipboard.setContent(content);
        });

        //清除按钮事件
        clear.setOnAction(action -> {
            source = null;
            image.setImage(null);
            imagePath = "";
            ratio = 0;
            width = 0;
            height = 0;
            resultArea.setText("");
            zoomlvl = 1;
            initx = 0;
            inity = 0;
            zoomLvl.setValue(1);
        });
    }

    //获取长宽、长宽比
    private void getWidthHeight() {
        width = (int) source.getWidth();
        height = (int) source.getHeight();
        ratio = source.getWidth() / source.getHeight();//获取长宽比
        if (500 / ratio < 500) {
            width = 500;
            height = (int) (500 / ratio);
        } else if (500 * ratio < 500) {
            height = 500;
            width = (int) (500 * ratio);
        } else {
            height = 500;
            width = 500;
        }
    }
}
```

### (3) 效果


![请添加图片描述](https://img-blog.csdnimg.cn/1f69bfdb4fba4a66afc83700eed9bee6.jpeg)


---

参考：
[JavaFX图片浏览并实现缩放](https://blog.csdn.net/m0_37649480/article/details/125941031)
[Zooming an Image in ImageView (javafx)](https://stackoverflow.com/questions/48687994/zooming-an-image-in-imageview-javafx)
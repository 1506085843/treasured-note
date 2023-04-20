本篇在上一篇安装了ffmpeg的基础上进行代码封装，Java里使用ProcessBuilder模拟命令行调用ffmpeg进行视频格式转换、音视频合并、播放、截图。

----

**FfmpegUtils封装类：**
ffplay、ffmpeg、ffprobe是安装的ffmpeg路径。

```java

import java.io.*;
import java.time.LocalDate;
import java.time.LocalTime;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

import static java.time.temporal.ChronoUnit.*;

/**
 * 利用ffmpeg进行音频视频操作，需先下载安装ffmpeg
 */
public class FfmpegUtils {
    /**
     * 下载的ffmpeg解压后的bin目录路径，可配置到环境变量通过配置文件读取
     */
    private static String ffplay = "D:\\Program File\\ffmpeg-4.3.1-2021-01-01-essentials_build\\bin\\ffplay.exe";
    private static String ffmpeg = "D:\\Program File\\ffmpeg-4.3.1-2021-01-01-essentials_build\\bin\\ffmpeg.exe";
    private static String ffprobe = "D:\\Program File\\ffmpeg-4.3.1-2021-01-01-essentials_build\\bin\\ffprobe.exe";
    /**
     * 提取的音频、合成的视频存放路径，不存在会自动创建
     */
    private static String saveMediaPath = "D:\\ffmpegMedia\\";

    /**
     * 保存音频、视频的临时文件夹，不存在会自动创建
     */
    private static String tempMediaPath = "D:\\ffmpegMedia\\temp\\";
    /**
     * 保存图片截图的文件夹，不存在会自动创建
     */
    private static String picturMediaPath = "D:\\ffmpegMedia\\pictur\\";


    static {
        //如果没有文件夹，则创建
        File saveMediaFile = new File(saveMediaPath);
        if (!saveMediaFile.exists() && !saveMediaFile.isDirectory()) {
            saveMediaFile.mkdirs();
        }
        File tempMediaFile = new File(tempMediaPath);
        if (!tempMediaFile.exists() && !tempMediaFile.isDirectory()) {
            tempMediaFile.mkdirs();
        }
        File picturMediaFile = new File(picturMediaPath);
        if (!picturMediaFile.exists() && !picturMediaFile.isDirectory()) {
            picturMediaFile.mkdirs();
        }
    }

    /**
     * 播放音频和视频
     *
     * @param resourcesPath 文件的路径
     */
    public static void playVideoAudio(String resourcesPath) {
        List<String> command = new ArrayList<>();
        command.add(ffplay);
        command.add("-window_title");
        String fileName = resourcesPath.substring(resourcesPath.lastIndexOf("\\") + 1);
        command.add(fileName);
        command.add(resourcesPath);
        //播放完后自动退出
        //command.add("-autoexit");
        commandStart(command);
    }

    /**
     * 播放音频和视频并指定循环次数
     *
     * @param resourcesPath 文件的路径
     * @param loop          循环播放次数
     */
    public static void playVideoAudio(String resourcesPath, int loop) {
        List<String> command = new ArrayList<>();
        command.add(ffplay);
        command.add("-window_title");
        String fileName = resourcesPath.substring(resourcesPath.lastIndexOf("\\") + 1);
        command.add(fileName);
        command.add(resourcesPath);
        command.add("-loop");
        command.add(String.valueOf(loop));
        //播放完后自动退出
        //command.add("-autoexit");
        commandStart(command);
    }

    /**
     * 播放音频和视频并指定宽、高、循环次数
     *
     * @param resourcesPath 文件的路径
     * @param weight        宽度
     * @param height        高度
     * @param loop          循环播放次数
     */
    public static void playVideoAudio(String resourcesPath, int weight, int height, int loop) {
        List<String> command = new ArrayList<>();
        command.add(ffplay);
        command.add("-window_title");
        String fileName = resourcesPath.substring(resourcesPath.lastIndexOf("\\") + 1);
        command.add(fileName);
        command.add(resourcesPath);
        command.add("-x");
        command.add(String.valueOf(weight));
        command.add("-y");
        command.add(String.valueOf(height));
        command.add("-loop");
        command.add(String.valueOf(loop));
        //播放完后自动退出
        //command.add("-autoexit");
        commandStart(command);
    }

    /**
     * 调用命令行执行
     *
     * @param command 命令行参数
     */
    public static void commandStart(List<String> command) {
        command.forEach(v -> System.out.print(v + " "));
        System.out.println();
        System.out.println();
        ProcessBuilder builder = new ProcessBuilder();
        //正常信息和错误信息合并输出
        builder.redirectErrorStream(true);
        builder.command(command);
        //开始执行命令
        Process process = null;
        try {
            process = builder.start();
            //如果你想获取到执行完后的信息，那么下面的代码也是需要的
            String line = "";
            BufferedReader br = new BufferedReader(new InputStreamReader(process.getInputStream()));
            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 调用命令行执行，并返回信息
     *
     * @param command 命令行参数
     */
    public static String getInfoStr(List<String> command) {
        command.forEach(v -> System.out.print(v + " "));
        System.out.println();
        System.out.println();
        ProcessBuilder builder = new ProcessBuilder();
        //正常信息和错误信息合并输出
        builder.redirectErrorStream(true);
        builder.command(command);
        //开始执行命令
        Process process = null;
        StringBuffer sb = new StringBuffer();
        try {
            process = builder.start();
            //如果你想获取到执行完后的信息，那么下面的代码也是需要的
            String line = "";
            BufferedReader br = new BufferedReader(new InputStreamReader(process.getInputStream()));
            while ((line = br.readLine()) != null) {
                System.out.println(line);
                sb.append(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return String.valueOf(sb);
    }


    /**
     * 从视频中提取音频为mp3
     *
     * @param videoResourcesPath 视频文件的路径
     */
    public static void getAudioFromVideo(String videoResourcesPath) {
        String fileName = videoResourcesPath.substring(videoResourcesPath.lastIndexOf("\\") + 1, videoResourcesPath.lastIndexOf("."));
        List<String> command = new ArrayList<>();
        command.add(ffmpeg);
        command.add("-i");
        command.add(videoResourcesPath);
        command.add(saveMediaPath + fileName + ".mp3");
        commandStart(command);
    }

    /**
     * 从视频中去除去音频并保存视频
     *
     * @param videoResourcesPath 视频文件的路径
     */
    public static void getVideoFromAudio(String videoResourcesPath) {
        List<String> command = new ArrayList<>();
        command.add(ffmpeg);
        command.add("-i");
        command.add(videoResourcesPath);
        command.add("-vcodec");
        command.add("copy");
        command.add("-an");
        command.add(saveMediaPath + videoResourcesPath.substring(videoResourcesPath.lastIndexOf("\\") + 1));
        commandStart(command);
    }


    /**
     * 无声视频+音频合并为一个视频
     * 若音频比视频长，画面停留在最后一帧，继续播放声音。
     *
     * @param videoResourcesPath 视频文件的路径
     * @param audioResourcesPath 音频文件的路径
     */
    public static void mergeSilent_VideoAudio(String videoResourcesPath, String audioResourcesPath) {
        List<String> command = new ArrayList<>();
        command.add(ffmpeg);
        command.add("-i");
        command.add(videoResourcesPath);
        command.add("-i");
        command.add(audioResourcesPath);
        command.add("-vcodec");
        command.add("copy");
        command.add("-acodec");
        command.add("copy");
        command.add(saveMediaPath + videoResourcesPath.substring(videoResourcesPath.lastIndexOf("\\") + 1));
        commandStart(command);
    }


    /**
     * 有声视频+音频合并为一个视频。
     * 若音频比视频长，画面停留在最后一帧，继续播放声音,
     * 若要以视频和音频两者时长短的为主，放开注解启用-shortest。
     *
     * @param videoResourcesPath 视频文件的路径
     * @param audioResourcesPath 音频文件的路径
     */
    public static void mergeVideoAudio(String videoResourcesPath, String audioResourcesPath) {
        List<String> command = new ArrayList<>();
        command.add(ffmpeg);
        command.add("-i");
        command.add(videoResourcesPath);
        command.add("-i");
        command.add(audioResourcesPath);
        command.add("-filter_complex");
        command.add("amix");
        command.add("-map");
        command.add("0:v");
        command.add("-map");
        command.add("0:a");
        command.add("-map");
        command.add("1:a");
        //-shortest会取视频或音频两者短的一个为准，多余部分则去除不合并
        //command.add("-shortest");
        command.add(saveMediaPath + videoResourcesPath.substring(videoResourcesPath.lastIndexOf("\\") + 1));
        commandStart(command);
    }


    /**
     * 多视频拼接合并
     *
     * @param videoResourcesPathList 视频文件路径的List
     */
    public static void mergeVideos(List<String> videoResourcesPathList) {
        //时间作为合并后的视频名
        String getdatatime = nowTime();
        //所有要合并的视频转换为ts格式存到videoList里
        List<String> videoList = new ArrayList<>();
        for (String video : videoResourcesPathList) {
            List<String> command = new ArrayList<>();
            command.add(ffmpeg);
            command.add("-i");
            command.add(video);
            command.add("-c");
            command.add("copy");
            command.add("-bsf:v");
            command.add("h264_mp4toannexb");
            command.add("-f");
            command.add("mpegts");
            String videoTempName = video.substring(video.lastIndexOf("\\") + 1, video.lastIndexOf(".")) + ".ts";
            command.add(tempMediaPath + videoTempName);
            commandStart(command);
            videoList.add(tempMediaPath + videoTempName);
        }

        List<String> command1 = new ArrayList<>();
        command1.add(ffmpeg);
        command1.add("-i");
        StringBuffer buffer = new StringBuffer("\"concat:");
        for (int i = 0; i < videoList.size(); i++) {
            buffer.append(videoList.get(i));
            if (i != videoList.size() - 1) {
                buffer.append("|");
            } else {
                buffer.append("\"");
            }
        }
        command1.add(String.valueOf(buffer));
        command1.add("-c");
        command1.add("copy");
        command1.add(saveMediaPath + "视频合并" + getdatatime + ".mp4");
        commandStart(command1);
    }

	/**
     * 多视频拼接合并。（mergeVideos1比上面的mergeVideos兼容性更好）
     *
     * @param videoResourcesPathList 视频文件路径的List
     */
    public static void mergeVideos1(List<String> videoResourcesPathList) {
        String videosPath= tempMediaPath+"videos.txt";
        try(PrintWriter writer = new PrintWriter(new BufferedWriter(new FileWriter(videosPath, false)))) {
            for (String video:videoResourcesPathList) {
                writer.println("file '"+video+"'");
            }
        }catch (IOException e) {
            System.err.println(e);
        }
        List<String> command = new ArrayList<>();
        command.add(ffmpeg);
        command.add("-f");
        command.add("concat");
        command.add("-safe");
        command.add("0");
        command.add("-i");
        command.add(videosPath);
        command.add("-c");
        command.add("copy");
        //时间作为合并后的视频名
        String getdatatime = nowTime();
        command.add(saveMediaPath + "视频合并" + getdatatime + ".mp4");
        commandStart(command);
    }

    /**
     * 多音频拼接合并为一个音频（在每个音频结尾追加另一个音频，即同一时间只播放一个音频）。
     *
     * @param audioResourcesPathList 音频文件路径的List
     */
    public static void mergeAudios(List<String> audioResourcesPathList) {
        //时间作为合并后的音频名
        String getdatatime = nowTime();
        List<String> command = new ArrayList<>();
        command.add(ffmpeg);
        command.add("-i");
        StringBuffer buffer = new StringBuffer("\"concat:");
        for (int i = 0; i < audioResourcesPathList.size(); i++) {
            buffer.append(audioResourcesPathList.get(i));
            if (i != audioResourcesPathList.size() - 1) {
                buffer.append("|");
            } else {
                buffer.append("\"");
            }
        }
        command.add(String.valueOf(buffer));
        command.add("-acodec");
        command.add("copy");
        command.add(saveMediaPath + "音频合并" + getdatatime + ".mp3");
        commandStart(command);
    }


    /**
     * 视频格式转换
     *
     * @param videoResourcesPath 视频文件的路径
     * @param format             要转换为的格式
     */
    public static void videoFormatConversion(String videoResourcesPath, String format) {
        String fileName = videoResourcesPath.substring(videoResourcesPath.lastIndexOf("\\") + 1, videoResourcesPath.lastIndexOf("."));
        List<String> command = new ArrayList<>();
        command.add(ffmpeg);
        command.add("-i");
        command.add(videoResourcesPath);
        command.add(saveMediaPath + fileName + "." + format);
        commandStart(command);
    }

    /**
     * 获取音频或视频信息
     *
     * @param videoAudioResourcesPath 音频或视频文件的路径
     */
    public static List<String> videoAudioInfo(String videoAudioResourcesPath) {
        List<String> command = new ArrayList<>();
        command.add(ffprobe);
        command.add("-i");
        command.add(videoAudioResourcesPath);
        //调用命令行获取视频信息
        String infoStr = getInfoStr(command);
        System.out.println(" ");

        String regexDuration = "Duration: (.*?), start: (.*?), bitrate: (\\d*) kb\\/s";
        String regexVideo = "Video: (.*?) tbr";
        String regexAudio = "Audio: (.*?), (.*?) Hz, (.*?) kb\\/s";
        List<String> list = new ArrayList<>();
        Pattern pattern = Pattern.compile(regexDuration);
        Matcher matcher = pattern.matcher(infoStr);
        if (matcher.find()) {
            list.add("视频/音频整体信息: ");
            list.add("视频/音频名称：" + videoAudioResourcesPath);
            list.add("开始时间：" + matcher.group(2));
            list.add("结束时间：" + matcher.group(1));
            list.add("比特率: " + matcher.group(3) + " kb/s");
            list.add("------------------------------------ ");
        }

        Pattern patternVideo = Pattern.compile(regexVideo);
        Matcher matcherVideo = patternVideo.matcher(infoStr);
        if (matcherVideo.find()) {
            String videoInfo = matcherVideo.group(1);
            String[] sp = videoInfo.split(",");
            list.add("视频流信息: ");
            list.add("视频编码格式: " + sp[0]);
            int ResolutionPosition = 2;
            if (sp[1].contains("(") && sp[2].contains(")")) {
                list.add("YUV: " + sp[1] + "," + sp[2]);
                ResolutionPosition = 3;
            } else if (sp[1].contains("(") && !sp[2].contains(")") && sp[3].contains(")")) {
                list.add("YUV: " + sp[1] + "," + sp[2] + "," + sp[3]);
                ResolutionPosition = 4;
            } else {
                list.add("YUV: " + sp[1]);
            }
            list.add("分辨率: " + sp[ResolutionPosition]);
            list.add("视频比特率: " + sp[ResolutionPosition + 1]);
            list.add("帧率: " + sp[ResolutionPosition + 2]);
            list.add("------------------------------------ ");
        }
        Pattern patternAudio = Pattern.compile(regexAudio);
        Matcher matcherAudio = patternAudio.matcher(infoStr);
        if (matcherAudio.find()) {
            list.add("音频流信息: ");
            list.add("音频编码格式: " + matcherAudio.group(1));
            list.add("采样率: " + matcherAudio.group(2) + " HZ");
            list.add("声道: " + matcherAudio.group(3).split(",")[0]);
            list.add("音频比特率: " + matcherAudio.group(3).split(",")[2] + " kb/s");
        }
        return list;
    }

    /**
     * 视频或音频剪切
     * 参考@link:https://zhuanlan.zhihu.com/p/27366331
     *
     * @param videoAudioResourcesPath 视频或音频文件的路径
     * @param startTime               开始时间
     * @param endTime                 结束时间
     */
    public static void cutVideoAudio(String videoAudioResourcesPath, String startTime, String endTime) {
        String fileName = videoAudioResourcesPath.substring(videoAudioResourcesPath.lastIndexOf("\\") + 1);
        //时间作为剪切后的视频名
        String getdatatime = nowTime();
        List<String> command = new ArrayList<>();
        command.add(ffmpeg);
        command.add("-ss");
        command.add(startTime);
        command.add("-t");
        command.add(calculationEndTime(startTime, endTime));
        command.add("-i");
        command.add(videoAudioResourcesPath);
        command.add("-c:v");
        command.add("libx264");
        command.add("-c:a");
        command.add("aac");
        command.add("-strict");
        command.add("experimental");
        command.add("-b:a");
        command.add("98k");
        command.add(saveMediaPath + "剪切视频" + getdatatime + fileName);
        commandStart(command);
    }


	/**
     * 视频裁剪大小尺寸（根据leftDistance和topDistance确定裁剪的起始点，再根据finallywidth和finallyHeight确定裁剪的宽和长）
     * 参考@link:https://www.cnblogs.com/yongfengnice/p/7095846.html
     *     @link:http://www.jq-school.com/show.aspx?id=737
     *
     * @param videoAudioResourcesPath 视频文件的路径
     * @param finallyWidth            裁剪后最终视频的宽度
     * @param finallyHeight           裁剪后最终视频的高度
     * @param leftDistance            开始裁剪的视频左边到y轴的距离（视频左下角为原点）
     * @param topDistance             开始裁剪的视频上边到x轴的距离（视频左下角为原点）
     *
     */
    public static void cropVideoSize(String videoAudioResourcesPath, String finallyWidth, String finallyHeight, String leftDistance, String topDistance) {
        String fileName = videoAudioResourcesPath.substring(videoAudioResourcesPath.lastIndexOf("\\") + 1);
        //时间作为剪切后的视频名
        String getdatatime = nowTime();
        List<String> command = new ArrayList<>();
        command.add(ffmpeg);
        command.add("-i");
        command.add(videoAudioResourcesPath);
        command.add("-vf");
        //获取视频信息得到原始视频长、宽
        List<String> list = FfmpegUtils.videoAudioInfo(videoAudioResourcesPath);
        String resolution = list.stream().filter(v -> v.contains("分辨率")).findFirst().get();
        String sp[] = resolution.split("x");
        String originalWidth = sp[0].substring(sp[0].indexOf(":") + 1).trim();
        String originalHeight = sp[1].substring(0, 4).trim();
        Integer cropStartWidth = Integer.parseInt(originalWidth) - Integer.parseInt(leftDistance);
        Integer cropStartHeight = Integer.parseInt(originalHeight) - Integer.parseInt(topDistance);
        command.add("crop=" + finallyWidth + ":" + finallyHeight + ":" + cropStartWidth + ":" + cropStartHeight);
        command.add(saveMediaPath + "裁剪视频" + getdatatime + fileName);
        commandStart(command);
    }


    /**
     * 计算两个时间的时间差
     *
     * @param startime 开始时间，如：00:01:09
     * @param endtime  结束时间，如：00:08:27
     * @return 返回xx:xx:xx形式，如：00:07:18
     */
    public static String calculationEndTime(String startime, String endtime) {
        LocalTime timeStart = LocalTime.parse(startime);
        LocalTime timeEnd = LocalTime.parse(endtime);
        long hour = HOURS.between(timeStart, timeEnd);
        long minutes = MINUTES.between(timeStart, timeEnd);
        long seconds = SECONDS.between(timeStart, timeEnd);
        minutes = minutes > 59 ? minutes % 60 : minutes;
        String hourStr = hour < 10 ? "0" + hour : String.valueOf(hour);
        String minutesStr = minutes < 10 ? "0" + minutes : String.valueOf(minutes);
        long getSeconds = seconds - (hour * 60 + minutes) * 60;
        String secondsStr = getSeconds < 10 ? "0" + getSeconds : String.valueOf(getSeconds);
        return hourStr + ":" + minutesStr + ":" + secondsStr;
    }


    /**
     * 视频截图
     *
     * @param videoResourcesPath 视频文件的路径
     * @param screenshotTime     截图的时间，如：00:01:06
     */
    public static void videoScreenshot(String videoResourcesPath, String screenshotTime) {
        //时间作为截图后的视频名
        String getdatatime = nowTime();
        List<String> command = new ArrayList<>();
        command.add(ffmpeg);
        command.add("-ss");
        command.add(screenshotTime);
        command.add("-i");
        command.add(videoResourcesPath);
        command.add("-f");
        command.add("image2");
        String fileName = videoResourcesPath.substring(videoResourcesPath.lastIndexOf("\\") + 1, videoResourcesPath.lastIndexOf("."));
        command.add(picturMediaPath + fileName + getdatatime + ".jpg");
        commandStart(command);
    }


    /**
     * 整个视频截图
     *
     * @param videoResourcesPath 视频文件的路径
     * @param fps                截图的速度。1则表示每秒截一张；0.1则表示每十秒一张；10则表示每秒截十张图片
     */
    public static void videoAllScreenshot(String videoResourcesPath, String fps) {
        /*//时间作为截图后的视频名
        String getdatatime = nowTime();*/
        List<String> command = new ArrayList<>();
        command.add(ffmpeg);
        command.add("-i");
        command.add(videoResourcesPath);
        command.add("-vf");
        command.add("fps=" + fps);
        String fileName = videoResourcesPath.substring(videoResourcesPath.lastIndexOf("\\") + 1, videoResourcesPath.lastIndexOf("."));
        command.add(picturMediaPath + fileName + "%d" + ".jpg");
        commandStart(command);
    }


    /**
     * 多图片+音频合并为视频
     *
     * @param pictureResourcesPath 图片文件路径(数字编号和后缀不要)。如：D:\ffmpegMedia\pictur\101-你也不必耿耿于怀1.jpg 和D:\ffmpegMedia\pictur\101-你也不必耿耿于怀2.jpg。只需传D:\ffmpegMedia\pictur\101-你也不必耿耿于怀
     * @param audioResourcesPath   音频文件的路径
     *  @param fps   帧率,每张图片的播放时间（数值越小则每张图停留的越长）。0.5则两秒播放一张，1则一秒播放一张，10则一秒播放十张
     */
    public static void pictureAudioMerge(String pictureResourcesPath, String audioResourcesPath,String fps) {
        //时间作为截图后的视频名
        String getdatatime = nowTime();
        List<String> command = new ArrayList<>();
        command.add(ffmpeg);
        command.add("-threads");
        command.add("2");
        command.add("-y");
        command.add("-r");
        //帧率
        command.add(fps);
        command.add("-i");
        command.add(pictureResourcesPath+"%d.jpg");
        command.add("-i");
        command.add(audioResourcesPath);
        command.add("-absf");
        command.add("aac_adtstoasc");
        //-shortest会取视频或音频两者短的一个为准，多余部分则去除不合并
        command.add("-shortest");
        String fileName = pictureResourcesPath.substring(pictureResourcesPath.lastIndexOf("\\") + 1);
        command.add(saveMediaPath +"视频合成"+ fileName + getdatatime + ".mp4");
        commandStart(command);
    }


    /**
     * 绘制音频波形图保存.jpg后缀可改为png
     *
     * @param audioResourcesPath 音频文件的路径
     */
    public static void audioWaveform(String audioResourcesPath) {
        List<String> command = new ArrayList<>();
        command.add(ffmpeg);
        command.add("-i");
        command.add(audioResourcesPath);
        command.add("-filter_complex");
        command.add("\"showwavespic=s=1280*240\"");
        command.add("-frames:v");
        command.add("1");
        String fileName = audioResourcesPath.substring(audioResourcesPath.lastIndexOf("\\") + 1, audioResourcesPath.lastIndexOf("."));
        //jpg可换为png
        command.add(saveMediaPath + fileName + fileName + ".jpg");
        commandStart(command);
    }


    /**
     * 两个音频混缩合并为一个音频（即同一时间播放两首音频）。
     * 音量参考：@link:https://blog.csdn.net/sinat_14826983/article/details/82975561
     *
     * @param audioResourcesPath1 音频1文件路径
     * @param audioResourcesPath1 音频2文件路径的
     */
    public static void mergeAudios(String audioResourcesPath1, String audioResourcesPath2) {
        //时间作为混缩后的音频名
        String getdatatime = nowTime();
        List<String> command = new ArrayList<>();
        command.add(ffmpeg);
        command.add("-i");
        command.add(audioResourcesPath1);
        command.add("-i");
        command.add(audioResourcesPath2);
        command.add("-filter_complex");
        command.add("amix=inputs=2:duration=longest");
        command.add(saveMediaPath + "音频混缩" + getdatatime + ".mp3");
        commandStart(command);
    }

    /**
     * 两个音频混缩合并为一个音频（即同一时间播放两首音频）。
     * 音量参考：@link:https://blog.csdn.net/sinat_14826983/article/details/82975561
     *
     * @param audioResourcesPath1 音频1文件路径
     * @param  number1           音频1的音量，如取 0.4 表示音量是原来的40%  ，取1.5表示音量是原来的150%
     * @param audioResourcesPath1 音频2文件路径的
     * @param  number2          音频2的音量，如取 0.4 表示音量是原来的40%  ，取1.5表示音量是原来的150%
     */
    public static void mergeAudios(String audioResourcesPath1,String number1, String audioResourcesPath2,String number2) {
        //时间作为混缩后的音频名
        String getdatatime = nowTime();
        List<String> command = new ArrayList<>();
        command.add(ffmpeg);
        command.add("-i");
        command.add(audioResourcesPath1);
        command.add("-i");
        command.add(audioResourcesPath2);
        command.add("-filter_complex");
        command.add("[0:a]volume=1"+number1+"[a1];[1:a]volume="+number2+"[a2];[a1][a2]amix=inputs=2:duration=longest");
        command.add(saveMediaPath + "音频混缩" + getdatatime + ".mp3");
        commandStart(command);
    }

    /**
     * 两个音频混缩合并为一个音频的不同声道（即一只耳机播放一个音频）。
     * 声道参考：@link:https://www.itranslater.com/qa/details/2583879740000044032
     *
     * @param audioResourcesPath1 音频1文件路径
     * @param audioResourcesPath1 音频2文件路径的
     */
    public static void mergeAudiosSoundtrack(String audioResourcesPath1, String audioResourcesPath2) {
        //时间作为混缩后的音频名
        String getdatatime = nowTime();
        List<String> command = new ArrayList<>();
        command.add(ffmpeg);
        command.add("-i");
        command.add(audioResourcesPath1);
        command.add("-i");
        command.add(audioResourcesPath2);
        command.add("-filter_complex");
        command.add("\"amerge=inputs=2,pan=stereo|c0<c0+c1|c1<c2+c3\"");
        command.add(saveMediaPath + "音频混缩" + getdatatime + ".mp3");
        commandStart(command);
    }

    /**
     * 获取当前时间，用于作为文件名
     */
    public static String nowTime() {
        DateTimeFormatter f3 = DateTimeFormatter.ofPattern("yyyy-MM-dd-HH-mm-ss");
        LocalDate nowdata = LocalDate.now();
        LocalTime nowTime = LocalTime.now();
        return nowdata.atTime(nowTime).format(f3);
    }
}

```

**测试：**

```java
import utils.FfmpegUtils;
import java.io.IOException;
import java.util.*;

public class Test {
    public static void main(String[] args) throws IOException {
        //视频播放
        FfmpegUtils.playVideoAudio("D:\\16-这几句真假声转换太好听了.mp4");
        //视频播放并指定循环次数
        FfmpegUtils.playVideoAudio("D:\\16-这几句真假声转换太好听了.mp4",1);
        //视频播放并指定宽高和循环次数
        FfmpegUtils.playVideoAudio("D:\\16-这几句真假声转换太好听了.mp4",400,700,1);
        //从视频中提取音频为mp3
        FfmpegUtils.getAudioFromVideo("D:\\16-这几句真假声转换太好听了.mp4");
        //从视频中提取视频为无声视频
        FfmpegUtils.getVideoFromAudio("D:\\16-这几句真假声转换太好听了.mp4");
        //无声视频+音频合并
        FfmpegUtils.mergeSilent_VideoAudio("D:\\507-#网愈云故事收藏馆.mp4","D:\\mp3\\16-这几句真假声转换太好听了.mp3");
        //格式转换
        FfmpegUtils.videoFormatConversion("D:\\507-#网愈云故事收藏馆.mp4","flv");
        //多视频拼接合并为一个mp4格式视频
        List<String> video = new ArrayList<>();
        video.add("D:\\16-这几句真假声转换太好听了.mp4");
        video.add("D:\\507-#网愈云故事收藏馆.mp4");
        video.add("D:\\101-你也不必耿耿于怀.mp4");
        //FfmpegUtils.mergeVideos(video);
        FfmpegUtils.mergeVideos1(video);
        //获取音频或视频信息
        List<String> list = FfmpegUtils.videoAudioInfo("D:\\ffmpegMedia\\16-这几句真假声转换太好听了.mp3");
        list.forEach(v -> System.out.println(v));
        //剪切视频或音频，startTime开始时间，endTime结束时间
        FfmpegUtils.cutVideoAudio("D:\\苏打绿带我走.mp4","00:00:00","00:01:05");
		//裁剪视频尺寸大小
        FfmpegUtils.cropVideoSize("D:\\张国荣.mp4", "600", "600", "720", "940");
        //有声视频+音频合并
        FfmpegUtils.mergeVideoAudio("D:\\507-#网愈云故事收藏馆.mp4","D:\\ffmpegMedia\\16-这几句真假声转换太好听了.mp3");
        //视频截图，screenshotTime是截图的时间
        FfmpegUtils.videoScreenshot("D:\\101-你也不必耿耿于怀.mp4","00:00:05");
        //视频完全截图，fps是截图的速度即多少秒截一张图
        FfmpegUtils.videoAllScreenshot("D:\\101-你也不必耿耿于怀.mp4","1");
        //多图片+音频合并为视频。0.5则两秒播放一张，1则一秒播放一张，10则一秒播放十张（数值越小则每张图停留的越长）
        FfmpegUtils.pictureAudioMerge("D:\\ffmpegMedia\\pictur\\101-你也不必耿耿于怀","D:\\ffmpegMedia\\101-你也不必耿耿于怀.mp3","0.5");
        //多音频拼接合并为一个mp3格式视频
        List<String> audio = new ArrayList<>();
        audio.add("D:\\ffmpegMedia\\16-这几句真假声转换太好听了.mp3");
        audio.add("D:\\ffmpegMedia\\忽然之间.mp3");
        audio.add("D:\\ffmpegMedia\\天空之城-李志.mp3");
        FfmpegUtils.mergeAudios(audio);
        //绘制音频波形图保存
        FfmpegUtils.audioWaveform("D:\\ffmpegMedia\\16-这几句真假声转换太好听了.mp3");
        //两个音频混缩合并为一个音频
        FfmpegUtils.mergeAudios("D:\\ffmpegMedia\\16-这几句真假声转换太好听了.mp3","D:\\ffmpegMedia\\天空之城-李志.mp3");
		//两个音频混缩合并为一个音频并调整两个音频的音量大小
		FfmpegUtils.mergeAudios("D:\\ffmpegMedia\\忽然之间.mp3","1","D:\\ffmpegMedia\\天空之城-李志.mp3","0.5");
        //两个音频混缩合并为一个音频的不同声道（即一只耳机播放一个音频）。
        FfmpegUtils.mergeAudiosSoundtrack("D:\\ffmpegMedia\\16-这几句真假声转换太好听了.mp3","D:\\ffmpegMedia\\天空之城-李志.mp3");
    }
}
```

上面测试中的一些用法已经基本满足基本使用。可以灵活组合运用。
**举个例子：**
现在网上的很多视频是分段成多段的ts视频，
我们可以写代码把所有的ts视频都下载下来，
然后转换成mp4格式，
最后合并为一个mp4视频格式。
![在这里插入图片描述](https://img-blog.csdnimg.cn/1d81302b05ff4e72ab989211fa7c5d20.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6KW_5YeJ55qE5oKy5Lyk,size_20,color_FFFFFF,t_70,g_se,x_16)

----
参考文章：
[Java中使用ProcessBuilder启动、管理应用程序](https://blog.csdn.net/qq_21383435/article/details/82709284?utm_medium=distribute.pc_relevant.none-task-blog-searchFromBaidu-2.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-searchFromBaidu-2.control)
[Singler line FFMPEG cmd to Merge Video /Audio and retain both audios](https://stackoverflow.com/questions/24804928/singler-line-ffmpeg-cmd-to-merge-video-audio-and-retain-both-audios)[工作记录--使用FFmpeg将一个视频文件中音频合成到另一个视频中](https://blog.csdn.net/weixin_34061482/article/details/94683349)
[FFmpeg精确时间截取视频文件](https://zhuanlan.zhihu.com/p/27366331)
[Black frames at beginning of video file when file cut](https://superuser.com/questions/1222801/black-frames-at-beginning-of-video-file-when-file-cut)
[FFmpeg：视频转码、剪切、合并、播放速调整](https://blog.csdn.net/angus_17/article/details/80696989)
[ffmpeg截取视频](https://www.cnblogs.com/magepi/p/10523250.html)
[YUV介绍](https://www.cnblogs.com/sddai/p/10302979.html)
[FFMpeg 常用命令格式转换，视频合成](https://www.cnblogs.com/xcjit/p/10831096.html)
[FFmpeg命令行工具学习(二)：播放媒体文件的工具ffplay](https://www.cnblogs.com/renhui/p/8458802.html)
[FFMPEG无损合并视频的多种方法](https://www.cnblogs.com/duoshou/articles/10255284.html)
[FFmpeg 使用命令整理 – 提取音频或视频、提取图片、格式转换等](https://www.hack520.com/668.html)
[使用ffmpeg合并多个视频文件](https://blog.csdn.net/winniezhang/article/details/89260841)
[ffmpeg常用音频处理](https://blog.csdn.net/sinat_14826983/article/details/82975561)
[如何使用ffmpeg叠加/缩混两个音频文件](https://www.itranslater.com/qa/details/2583879740000044032)
[ffmpeg 视频截图](https://blog.csdn.net/ternence_hsu/article/details/92980451)
[FFmpeg将多张图片合成视频](http://blog.chinaunix.net/uid-27875-id-5784392.html)
[ffmpeg调整缩放裁剪视频的基础知识](https://www.cnblogs.com/yongfengnice/p/7095846.html)
[ffmpeg视频切片、缩放与裁剪输出m3u8流](http://www.jq-school.com/show.aspx?id=737)
[FFmpeg处理音视频命令汇总](https://blog.csdn.net/Jason_Lewis/article/details/90696878)
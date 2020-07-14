---
title: 'Deploying YOLO Object Detection Model With DeepLearning 4 Java & Apache Flink in Kubernets Cluster Using Raspberry PI!'
date: 2019-10-14
permalink: /posts/2019/10/blog-post-4/
tags:
  - Apache Flink
  - Deep Learning
  - Object Detection
---


> I was trying to test how object detection model working on kuberenets cluster with apache flink. As i love to do everything in java rather than python, i wanted to give a shot for DeepLearning4j API. This API has been built for running various deep learning model on top of Java. it works pretty good. I set up a kubernets cluster to run the Apache Flink and deploy the YOLO model with flink configuration to see how it works. Let's move on to step by step process. 

## Kubernets Cluster
![raspi stack](/images/raspi.jpg)


For Kuberenets cluster i have used severel Raspberry Pi to make the cluster works. First of all, i have used HypriotOS for firing up the RasPi's. It has build in docker support. I have kept all the RasPI's in same network, so that all the device can communicate with each other easily. After setting up the netwrok i moved for initializing the kubernets cluster. To initialize the kuberenets cluster you have to go through number of commands. You can get these commands from Kubernets official website or you can use from here.


>Note: For master node your RasPI has to have at least 2 GB of RAM and 2 CPU Core. Otherwise it will not work from Kubernets version 1.10.

As i have used normal RasPI with 1GB of RAM and 1 CPU, i have initialiazed master node with kuberenets version 1.9.0. This is the last kuberenets version which works with 1GB RAM Master Node.

Before initialization you need to install necessary dependencies like docker, kubelet, kubeadm, kubectl. 

> curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update -q
>sudo apt-get install -qy --force-yes kubeadm=1.9.6-00 kubectl=1.9.6-00 kubelet=1.9.6-00

>sudo aptitude install docker-ce=17.12.1~ce-0~raspbian

Make sure to do the swapoff "sudo swapoff -a" if you want to make cluster using ubuntu machine.

### Initialize the Master Node:
>kubeadm init --pod-network-cidr=172.16.0.0/16 --apiserver-advertise-address=192.168.1.2

In above command , you have to give your master node ip address , though it is not necessary. As i ahve used flannel network , i have mentioned this parameters. After entering this command you will get the kuberenets initialization message with join command for slave nodes. With this join command you can join any number of slave nodes you want. We will talk about this later.

### Configure the master node:

You will get some commands after initializing the master node, using thiscommand you can configure the master node.

> mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

For communicating between the pods in cluster we need to configure the network. Later I have used weave net for pod networking rather than flannel for few reasons. Though you can use flannel also.
>Kubernet pod networking activation: 
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"


### Now we can join other slave nodes to the cluster with master nodes.
For that we need to again install all the necessary dependencies like docker, kubeadm as i mentioned the command earlier. Run these command again to all of your slave nodes. Then run the join command to you every slave nodes...

>kubeadm join --token fd1017.8591d160fa6cec7c 192.168.1.3:6443 --discovery-token-ca-cert-hash sha256:875cc0af9878ef860f2d36c916b31f862b458f5455d1a73488856fb6b108cb7c

You can recreate your join command any time by using this command.
>kubeadm token create --print-join-command

Now you will see the message that node has joined to master node in the cluster after running the join command in every slave node. 

Now if run the follwing command in kubernets master node, you will be able to see all the nodes in the cluster.
>kubectl get nodes

![Nodes](/images/nodes.jpg)

You can algo get all the pods using following command..
>kubectl get pods
![Pods](/images/pods.jpg)
If you want to see the log or want to see the details of pods,
>kubectl log "pod-name"
>kubectl describe pod "pod-name"

If you want to bring the kuberents dashboard then you can run following deployment command to bring the dashboard pod online. 
>kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

To access Dashboard from your local workstation you must create a secure channel to your Kubernetes cluster. Run the following command:
>kubectl proxy 

Now access Dashboard at:

>http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

To login to the dashboard you need to create some token or user id. The follwoing you can generate the token to login to the dashbaord.

 >kubectl create serviceaccount cluster-admin-dashboard-sa
 >kubectl create clusterrolebinding cluster-admin-dashboard-sa \
  --clusterrole=cluster-admin \
  --serviceaccount=default:cluster-admin-dashboard-sa
>kubectl get secret | grep cluster-admin-dashboard-sa
>kubectl describe secret “cluster-admin-dashboard-sa-token-6xm8l(output from previous command)”

Use the generated token to login to the dashboard.

### To delete the dashboard , you can use following commands...

>kubectl delete deployment kubernetes-dashboard --namespace=kube-system 
kubectl delete service kubernetes-dashboard  --namespace=kube-system 
kubectl delete role kubernetes-dashboard-minimal --namespace=kube-system 
kubectl delete rolebinding kubernetes-dashboard-minimal --namespace=kube-system
kubectl delete sa kubernetes-dashboard --namespace=kube-system 
kubectl delete secret kubernetes-dashboard-certs --namespace=kube-system
>kubectl delete secret kubernetes-dashboard-key-holder --namespace=kube-system


now we have the working kuberenets clusters with multiple slave nodes. now we will move on the Flink YOLO Application to run on the cluster.
## Flink with DeepLearning4J

Fire up the IDE like eclipse, netbeans or IntilligentIDE... Create a Maven project with your prefer name. You will get the POM file for set the maven dependencies. You have to add the FLINK dependencies and DeepLearning4J dependencies from maven website. My pom format..


```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>dlsp-fl</artifactId>
        <groupId>ods.bbdc</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <groupId>ods.bbdc.dlsp-fl</groupId>
    <artifactId>objdectectionUD</artifactId>

    <properties>
        <jar.fileName>ods.bbdc.dlsp-fl.objdetectionUD</jar.fileName>
        <jar.libDir>libs/</jar.libDir>
        <dl4j.version>1.0.0-beta</dl4j.version>
        <ffmpeg.version>3.2.1-1.3</ffmpeg.version>
        <javacv.version>1.4.1</javacv.version>
    </properties>


    <build>
        <finalName>${jar.fileName}</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.1.1</version>

                <configuration>
                    <outputDirectory>${HOME}/bbdc/</outputDirectory>
                    <archive>
                        <manifest>
                            <mainClass>p1.RunYolo1</mainClass>
                            <addClasspath>true</addClasspath>
                            <classpathPrefix>${jar.libDir}</classpathPrefix>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>

            
            </plugin>
            -->
        </plugins>
    </build>


    <dependencies>

        <dependency>
            <groupId>org.nd4j</groupId>
            <artifactId>nd4j-native-platform</artifactId>
            <version>${dl4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.deeplearning4j</groupId>
            <artifactId>deeplearning4j-core</artifactId>
            <version>${dl4j.version}</version>
        </dependency>
           </dependencies>

</project>



```

This POM file will create a apache flink jar file with all the necessary dependencies to run in the flink kuberenets cluster.

Below i have provided all the classes needed to run the program. You can compile it with follwing command in terminal in the project folder. It will create jar file.

> maven clean install

Runyolo class starts th program and sends the input parameter to VideoPlayer class where the thread starts and sends the image frame to YOLO1 class to run the detector to detect the object and it returns the detected object list. For getting the object as a flink out put you need to Serialzie the detectedobject list as this is not serializable in native YOLO library. The flink configuration for execution has been added to the code.Read through the code to undrstand properly.
>The original code has been taken from RamonTech and modified to run in the flink environment.
### RunYolo1.java Class


```java
package p1;

import java.io.File;
import java.util.List;

import javax.management.loading.PrivateClassLoader;

import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.utils.ParameterTool;
import org.deeplearning4j.nn.layers.objdetect.DetectedObject;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.apache.flink.api.java.DataSet;


public class RunYolo1 {

    private static String videoFileName;

    static {
        videoFileName = System.getProperty("user.dir") + "/videoSample2.mp4";
    }
    public static void main(String[] args) throws Exception {
        final ParameterTool params = ParameterTool.fromArgs(args);
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        env.getConfig().setGlobalJobParameters(params);
        Yolo1 yolo=null;

        //Yolo yolo = null;
        Logger log = LoggerFactory.getLogger(RunYolo.class);

        VideoPlayer videoPlayer = new VideoPlayer();
        videoPlayer.startRealTimeVideoDetection(env,
                params,
                videoFileName,
                Speed.FAST, true);
    }



```


## Yolo1.java class

```java
package p1;
import lombok.extern.slf4j.Slf4j;

import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.utils.ParameterTool;
import org.apache.flink.core.fs.FileSystem;
import org.bytedeco.javacv.Frame;
import org.bytedeco.javacv.Java2DFrameConverter;
import org.datavec.image.loader.NativeImageLoader;
import org.deeplearning4j.nn.graph.ComputationGraph;
import org.deeplearning4j.nn.layers.objdetect.DetectedObject;
import org.deeplearning4j.nn.layers.objdetect.Yolo2OutputLayer;
import org.deeplearning4j.nn.layers.objdetect.YoloUtils;
import org.deeplearning4j.zoo.model.TinyYOLO;
import org.deeplearning4j.zoo.model.YOLO2;
import org.nd4j.linalg.api.ndarray.INDArray;
//import org.nd4j.linalg.dataset.DataSet;
import org.nd4j.linalg.dataset.api.preprocessor.ImagePreProcessingScaler;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.java.DataSet;
import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;

import java.io.File;
import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Stack;
import java.util.stream.Collectors;

import static org.bytedeco.javacpp.opencv_core.*;
import static org.bytedeco.javacpp.opencv_imgproc.putText;
import static org.bytedeco.javacpp.opencv_imgproc.rectangle;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Slf4j
public class Yolo1 {
    private static Logger log = LoggerFactory.getLogger(Yolo.class);

    private static final double DETECTION_THRESHOLD = 0.5;
    //more accurate but slower
    public static ComputationGraph YOLO_V2_MODEL_PRE_TRAINED;
    //less accurate but faster
    public static final ComputationGraph TINY_YOLO_V2_MODEL_PRE_TRAINED;

    static {
        try {
            TINY_YOLO_V2_MODEL_PRE_TRAINED = (ComputationGraph) TinyYOLO.builder().build().initPretrained();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    private final Stack<Frame> stack = new Stack();
    private final Speed selectedSpeed;

    public static volatile List<DetectedObjectFlinkSerializable> predictedObjects;
    //private volatile List<DetectedObject> predictedObjects;
    private HashMap<Integer, String> map;
    private HashMap<String, String> groupMap;
    private ComputationGraph model;
    public static DataSet<DetectedObjectFlinkSerializable> pc;


    public Yolo1(ExecutionEnvironment env, ParameterTool params, Speed selectedSpeed, boolean yolo) throws IOException {
        this.selectedSpeed = selectedSpeed;

        if (yolo) {
            //real yolo v2
            if (YOLO_V2_MODEL_PRE_TRAINED == null) {
                YOLO_V2_MODEL_PRE_TRAINED = (ComputationGraph) YOLO2.builder().build().initPretrained();
            }
            prepareYOLOLabels();
        } else {
            //tiny yolo
            prepareTinyYOLOLabels();
        }
        model = getSelectedModel(yolo);
        warmUp(selectedSpeed);
    }

    private void warmUp(Speed selectedSpeed) throws IOException {
//        Yolo2OutputLayer outputLayer = (Yolo2OutputLayer) model.getOutputLayer(0);
//        BufferedImage read = ImageIO.read(new File("/home/hasan/files/sample.jpg"));
//        INDArray indArray = prepareImage(read, selectedSpeed.width, selectedSpeed.height);
//        INDArray results = model.outputSingle(indArray);
//        outputLayer.getPredictedObjects(results, DETECTION_THRESHOLD);
    }

    public void push(Frame matFrame) {
        stack.push(matFrame);
    }

    public void drawBoundingBoxesRectangles(Frame frame, Mat matFrame) {
        if (invalidData(frame, matFrame)) return;

        ArrayList<DetectedObject> detectedObjects = new ArrayList<>(predictedObjects);
        YoloUtils.nms(detectedObjects, 0.5);
        for (DetectedObject detectedObject : detectedObjects) {
            createBoundingBoxRectangle(matFrame, frame.imageWidth, frame.imageHeight, detectedObject);
        }

    }

    private boolean invalidData(Frame frame, Mat matFrame) {
        return predictedObjects == null || matFrame == null || frame == null;
    }

    public void predictBoundingBoxes(ExecutionEnvironment env, ParameterTool params) throws Exception {
        long start = System.currentTimeMillis();

        Yolo2OutputLayer outputLayer = (Yolo2OutputLayer) model.getOutputLayer(0);
        INDArray indArray = prepareImage(stack.pop(), selectedSpeed.width, selectedSpeed.height);
        log.info("stack of frames size " + stack.size());

        if (indArray == null) {
            return;
        }

        INDArray results = model.outputSingle(indArray);

        if (results == null) {
            return;
        }

        List<DetectedObject> preObject = outputLayer.getPredictedObjects(results, DETECTION_THRESHOLD);
        predictedObjects = preObject.stream()
                                    .map(o -> (new DetectedObjectFlinkSerializable(o)))
                                    .collect(Collectors.toList());

        pc = env.fromCollection(predictedObjects);
        pc.print();

        //pc.writeAsText("/home/hasan/files/predictedclass1", FileSystem.WriteMode.OVERWRITE);
        //System.out.println("predicted objects:"+predictedObjects);

        log.info("stack of predictions size " + predictedObjects.size());
        log.info("Prediction time " + (System.currentTimeMillis() - start) / 1000d);


        env.execute();
    }
    public List<DetectedObjectFlinkSerializable> result() {
        return predictedObjects;
    }

    private ComputationGraph getSelectedModel(boolean yolo) {
        ComputationGraph model;
        if (yolo) {
            model = YOLO_V2_MODEL_PRE_TRAINED;
        } else {
            model = TINY_YOLO_V2_MODEL_PRE_TRAINED;
        }
        return model;
    }

    private INDArray prepareImage(Frame frame, int width, int height) throws IOException {
        if (frame == null || frame.image == null) {
            return null;
        }
        BufferedImage convert = new Java2DFrameConverter().convert(frame);
        return prepareImage(convert, width, height);
    }

    private INDArray prepareImage(BufferedImage convert, int width, int height) throws IOException {
        NativeImageLoader loader = new NativeImageLoader(height, width, 3);
        ImagePreProcessingScaler imagePreProcessingScaler = new ImagePreProcessingScaler(0, 1);

        INDArray indArray = loader.asMatrix(convert);
        if (indArray == null) {
            return null;
        }
        imagePreProcessingScaler.transform(indArray);
        return indArray;
    }

    private void prepareYOLOLabels() {
        prepareLabels(COCO_CLASSES);
    }

    private void prepareLabels(String[] coco_classes) {
        if (map == null) {
            groupMap = new HashMap<>();
            groupMap.put("car", "Car");
            groupMap.put("bus", "Car");
            groupMap.put("truck", "Car");
            int i = 0;
            map = new HashMap<>();
            for (String s1 : coco_classes) {
                map.put(i++, s1);
                groupMap.putIfAbsent(s1, s1);
            }
        }
    }

    private void prepareTinyYOLOLabels() {
        prepareLabels(TINY_COCO_CLASSES);
    }

    private void createBoundingBoxRectangle(Mat file, int w, int h, DetectedObject obj) {

        double[] xy1 = obj.getTopLeftXY();
        double[] xy2 = obj.getBottomRightXY();
        int predictedClass = obj.getPredictedClass();

        int x1 = (int) Math.round(w * xy1[0] / selectedSpeed.gridWidth);
        int y1 = (int) Math.round(h * xy1[1] / selectedSpeed.gridHeight);
        int x2 = (int) Math.round(w * xy2[0] / selectedSpeed.gridWidth);
        int y2 = (int) Math.round(h * xy2[1] / selectedSpeed.gridHeight);
        rectangle(file, new Point(x1, y1), new Point(x2, y2), Scalar.RED);
        putText(file, groupMap.get(map.get(predictedClass)), new Point(x1 + 2, y2 - 2), FONT_HERSHEY_DUPLEX, 1, Scalar.GREEN);
        //log.info(groupMap.get(map.get(predictedClass)));
    }


    private final String[] COCO_CLASSES = {"person", "bicycle", "car", "motorbike", "aeroplane", "bus", "train",
            "truck", "boat", "traffic light", "fire hydrant", "stop sign", "parking meter", "bench", "bird", "cat",
            "dog", "horse", "sheep", "cow", "elephant", "bear", "zebra", "giraffe", "backpack", "umbrella", "handbag",
            "tie", "suitcase", "frisbee", "skis", "snowboard", "sports ball", "kite", "baseball bat", "baseball glove",
            "skateboard", "surfboard", "tennis racket", "bottle", "wine glass", "cup", "fork", "knife", "spoon", "bowl",
            "banana", "apple", "sandwich", "orange", "broccoli", "carrot", "hot dog", "pizza", "donut", "cake", "chair",
            "sofa", "pottedplant", "bed", "diningtable", "toilet", "tvmonitor", "laptop", "mouse", "remote", "keyboard",
            "cell phone", "microwave", "oven", "toaster", "sink", "refrigerator", "book", "clock", "vase", "scissors",
            "teddy bear", "hair drier", "toothbrush"};
    private final String[] TINY_COCO_CLASSES = {"aeroplane", "bicycle", "bird", "boat", "bottle", "bus", "car",
            "cat", "chair", "cow", "diningtable", "dog", "horse", "motorbike", "person", "pottedplant",
            "sheep", "sofa", "train", "tvmonitor"};
}

```

## Videoplayer.java class

```java
package p1;

import lombok.extern.slf4j.Slf4j;

import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.utils.ParameterTool;
import org.bytedeco.javacpp.opencv_core;
import org.bytedeco.javacv.FFmpegFrameGrabber;
import org.bytedeco.javacv.Frame;
import org.bytedeco.javacv.FrameGrabber;
import org.bytedeco.javacv.OpenCVFrameConverter;

import java.io.File;
import java.util.concurrent.atomic.AtomicInteger;

import static org.bytedeco.javacpp.opencv_highgui.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Slf4j
public class VideoPlayer {
	private static Logger log = LoggerFactory.getLogger(VideoPlayer.class);

    private static final String AUTONOMOUS_DRIVING = "Autonomous Driving(TU Berlin)";
    private String windowName;
    private volatile boolean stop = false;
    private Yolo yolo;
    private final OpenCVFrameConverter.ToMat converter = new OpenCVFrameConverter.ToMat();
    public final static AtomicInteger atomicInteger = new AtomicInteger();


    public void startRealTimeVideoDetection(ExecutionEnvironment env, ParameterTool params, String videoFileName, Speed selectedIndex, boolean yoloModel) throws java.lang.Exception {
        log.info("Start detecting video " + videoFileName);
        
        int id = atomicInteger.incrementAndGet();
        windowName = AUTONOMOUS_DRIVING + id;
        log.info(windowName);
        yolo = new Yolo(env,params,selectedIndex, yoloModel);
        startYoloThread(env,params);
        runVideoMainThread(videoFileName, converter);
    }

    private void runVideoMainThread(String videoFileName, OpenCVFrameConverter.ToMat toMat) throws FrameGrabber.Exception {
    	File videoFile = new File(videoFileName);
        FFmpegFrameGrabber grabber = initFrameGrabber(videoFileName);
        while (!stop) {
            Frame frame = grabber.grab();
            if (frame == null) {
                log.info("Stopping");
                stop();
                break;
            }
            if (frame.image == null) {
                continue;
            }
            yolo.push(frame);
            opencv_core.Mat mat = toMat.convert(frame);
            yolo.drawBoundingBoxesRectangles(frame, mat);
            imshow(windowName, mat);
            char key = (char) waitKey(20);
            // Exit this loop on escape:
            if (key == 27) {
                stop();
                break;
            }
        }
    }

    private FFmpegFrameGrabber initFrameGrabber(String videoFileName) throws FrameGrabber.Exception {
        FFmpegFrameGrabber grabber = new FFmpegFrameGrabber(videoFileName);
        grabber.start();
        return grabber;
    }

    private void startYoloThread(ExecutionEnvironment env, ParameterTool params) {
        Thread thread = new Thread(() -> {
            while (!stop) {
                try {
                    yolo.predictBoundingBoxes(env,params);
                } catch (Exception e) {
                    //ignoring a thread failure
                    //it may fail because the frame may be long gone when thread get chance to execute
                }
            }
            yolo = null;
            log.info("YOLO Thread Exit");
        });
        thread.start();
    }

    public void stop() {
        if (!stop) {
            stop = true;
            destroyAllWindows();
        }
    }
}


```

## Speed.java class
```java
package p1;

public enum Speed {

    FAST("Real-Time but low accuracy", 224, 224, 7, 7),
    MEDIUM("Almost Real-time and medium accuracy", 416, 416, 13, 13),
    SLOW("Slowest but high accuracy", 608, 608, 19, 19);

    private final String name;

    public final int width;
    public final int height;
    public final int gridWidth;
    public final int gridHeight;

    public String getName() {
        return name;
    }

    Speed(String name, int width, int height, int gridWidth, int gridHeight) {

        this.name = name;
        this.width = width;
        this.height = height;
        this.gridWidth = gridWidth;
        this.gridHeight = gridHeight;
    }



    @Override
    public String toString() {
        return name;
    }
}


```

## DetectedObjectSerializeable.java class

```java
package p1;

import org.deeplearning4j.nn.layers.objdetect.DetectedObject;
import org.nd4j.linalg.api.ndarray.INDArray;

import java.io.Serializable;

public class DetectedObjectFlinkSerializable extends DetectedObject implements Serializable {

    public DetectedObjectFlinkSerializable(DetectedObject detectedObject){
        this(detectedObject.getExampleNumber(),
             detectedObject.getCenterX(),
             detectedObject.getCenterY(),
             detectedObject.getHeight(),
             detectedObject.getWidth(),
             detectedObject.getClassPredictions(),
             detectedObject.getConfidence());

    }

    public DetectedObjectFlinkSerializable(int exampleNumber, double centerX, double centerY, double width, double height, INDArray classPredictions, double confidence) {
        super(exampleNumber, centerX, centerY, width, height, classPredictions, confidence);
    }


}


```
## Run in the kubernets cluster

To run the generated jar file into the flink cluster in Kuberenets, you need to deploy flink jobmanager and task manager pods into kuberenets cluster. As this program need lot of heap memory to run, you need modified flink cluster as the normal flink docker image has 1GB heap memory but we need minimum 4gb heap memory. I have added the deployment.yaml file for initializing the flink deployment.

For jobamanger-Service.yaml:
>kubectl create -f jobmanager-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: flink-jobmanager
spec:
  ports:
  - name: rpc
    port: 6123
  - name: blob
    port: 6124
  - name: query
    port: 6125
  - name: ui
    port: 8081
  selector:
    app: flink
    component: jobmanager


```

For jobamanger-deployment.yaml:
kubectl create -f jobmanager-deployment.yaml

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: flink-jobmanager
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: flink
        component: jobmanager
    spec:
      containers:
      - name: jobmanager
        image: flink:latest
        args:
        - jobmanager
        ports:
        - containerPort: 6123
          name: rpc
        - containerPort: 6124
          name: blob
        - containerPort: 6125
          name: query
        - containerPort: 8081
          name: ui
        volumeMounts:
        - name: config
          mountPath: /opt/flink/conf/flink-conf.yaml
          subPath: flink-conf.yaml
      volumes:
      - name: config
        configMap:
          name: flink-config

```

For taskmanger-deployment.yaml:
>kubectl create -f taskmanager-deployment.yaml

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: flink-taskmanager
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: flink
        component: taskmanager
    spec:
      containers:
      - name: taskmanager
        image: flink:latest
        args:
        - taskmanager
        - "-Dtaskmanager.host=$(K8S_POD_IP)"
        ports:
        - containerPort: 6121
          name: data
        - containerPort: 6122
          name: rpc
        - containerPort: 6125
          name: query
        volumeMounts:
        - name: config
          mountPath: /opt/flink/conf/flink-conf.yaml
          subPath: flink-conf.yaml
      volumes:
      - name: config
        configMap:
          name: flink-config
```
For modifying the flink.yaml file to match up with the necessary heap memory, here is the config file i created and used..
>kubectl create -f config.yaml
config.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: flink-config
data:
  flink-conf.yaml: |
    blob.server.port: 6124
    jobmanager.rpc.port: 6123
    jobmanager.rpc.address: flink-jobmanager
    jobmanager.heap.size: 4096m
    taskmanager.heap.size: 4096m
    taskmanager.numberOfTaskSlots: 2
```

After deploying all the yaml file, you will get the flink running on the kuberenets cluster. You will see one jobmanager and two taskmanager pods in running state.

Then you can access the flink UI from browser. For that, you need to open up the proxy through terminal using follwoing command..
>kubectl proxy

and browse to the follwink link. 
>http://localhost:8001/api/v1/namespaces/default/services/flink-jobmanager:ui/proxy

![Flink UI](/images/ui.jpg)

Then you can submit the job using flink webUI. just upload the object detection jar file and run the jar file using necessary parameters or without any parameter. You can check the log file of the taskmanager to see the output.

> run in the flink cluster in kuberenets, you need to copy the input video file in all the flink taskmanager pods and set the path properly in the code.

To copy file into the flink taskmanager pods..

>kubectl cp "path/of/the/file" flink-taskmanager-blablabla:./

This is how i run the object detection model in kuberenets cluster using flink and deep learning, for question, please email me at hasan.alive@gmail.com.

Thanks

------

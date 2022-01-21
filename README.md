# 9206Autonomous
Here is one of 9206 iRobotics' Autonomous for the 2021-2022 FTC Freight Frenzy competition.
The code attached is for the Red side. 
In chronological order, here is what the autonomous does, you can modify this to fit your needs
1. Detect the location of a capstone on the 3 markers closest to the warehouse.
2. Raise a claw to the location of the correct level on the shipping hub
3. Go forward to the shipping hub and drop the pre-loaded block onto the shipping hub.
4. Back up against the wall.
5. Turn 90 degrees and drive into the warehouse.
6. End.

package org.firstinspires.ftc.teamcode;

import com.qualcomm.robotcore.eventloop.opmode.Autonomous;
import com.qualcomm.robotcore.eventloop.opmode.LinearOpMode;
import com.qualcomm.robotcore.hardware.CRServo;
import com.qualcomm.robotcore.hardware.DcMotor;
import com.qualcomm.robotcore.hardware.DcMotorSimple;
import java.util.List;
import org.firstinspires.ftc.robotcore.external.hardware.camera.WebcamName;
import org.firstinspires.ftc.robotcore.external.navigation.AxesOrder;
import org.firstinspires.ftc.robotcore.external.navigation.VuforiaCurrentGame;
import org.firstinspires.ftc.robotcore.external.navigation.VuforiaLocalizer;
import org.firstinspires.ftc.robotcore.external.tfod.Recognition;
import org.firstinspires.ftc.robotcore.external.tfod.TfodCurrentGame;

@Autonomous(name = "CameraSHDuckTapeRed (Blocks to Java)")
public class CameraSHDuckTapeRed extends LinearOpMode {

  private VuforiaCurrentGame vuforiaFreightFrenzy;
  private TfodCurrentGame tfodFreightFrenzy;
  private CRServo claw;
  private DcMotor RightFront;
  private DcMotor LeftFront;
  private DcMotor RightBack;
  private DcMotor LeftBack;
  private DcMotor lift;
  private DcMotor spin;

  float CapStoneLocation;

  /**
   * Describe this function...
   */
  private void Initialize_Camera() {
    CapStoneLocation = false;
    // Sample TFOD Op Mode
    // Initialize Vuforia.
    vuforiaFreightFrenzy.initialize(
        "", // vuforiaLicenseKey
        hardwareMap.get(WebcamName.class, "Camera"), // cameraName
        "", // webcamCalibrationFilename
        false, // useExtendedTracking
        false, // enableCameraMonitoring
        VuforiaLocalizer.Parameters.CameraMonitorFeedback.NONE, // cameraMonitorFeedback
        0, // dx
        0, // dy
        0, // dz
        AxesOrder.XZY, // axesOrder
        90, // firstAngle
        90, // secondAngle
        0, // thirdAngle
        true); // useCompetitionFieldTargetLocations
    // Set min confidence threshold to 0.7
    tfodFreightFrenzy.initialize(vuforiaFreightFrenzy, (float) 0.5, true, true);
    // Initialize TFOD before waitForStart.
    // Init TFOD here so the object detection labels are visible
    // in the Camera Stream preview window on the Driver Station.
    tfodFreightFrenzy.activate();
    telemetry.update();
  }

  /**
   * Describe this function...
   */
  private void Camera2() {
    List<Recognition> recognitions;
    int index;
    Recognition recognition;

    for (int count = 0; count < 300; count++) {
      // Put loop blocks here.
      // Get a list of recognitions from TFOD.
      recognitions = tfodFreightFrenzy.getRecognitions();
      if (recognitions.size() != 0) {
        index = 0;
        // Iterate through list and call a function to
        // display info for each recognized object.
        for (Recognition recognition_item : recognitions) {
          recognition = recognition_item;
          if (!"Marker".equals(recognition.getLabel())) {
            CapStoneLocation = (recognition.getLeft() + recognition.getRight()) / 2;
          }
          // Display info.
          // Increment index.
          index = index + 1;
        }
      }
      telemetry.update();
      if (false != CapStoneLocation) {
        break;
      }
    }
    tfodFreightFrenzy.deactivate();
    vuforiaFreightFrenzy.deactivate();
  }

  /**
   * This function is executed when this Op Mode is selected from the Driver Station.
   */
  @Override
  public void runOpMode() {
    int Team;

    vuforiaFreightFrenzy = new VuforiaCurrentGame();
    tfodFreightFrenzy = new TfodCurrentGame();
    claw = hardwareMap.get(CRServo.class, "claw");
    RightFront = hardwareMap.get(DcMotor.class, "Right Front");
    LeftFront = hardwareMap.get(DcMotor.class, "Left Front");
    RightBack = hardwareMap.get(DcMotor.class, "Right Back");
    LeftBack = hardwareMap.get(DcMotor.class, "Left Back");
    lift = hardwareMap.get(DcMotor.class, "lift");
    spin = hardwareMap.get(DcMotor.class, "spin");

    Team = -1;
    // Red = -1; Blue = 1
    for (int count2 = 0; count2 < 1000; count2++) {
      claw.setDirection(DcMotorSimple.Direction.REVERSE);
      claw.setPower(0.5);
      telemetry.update();
    }
    Initialize_Camera();
    waitForStart();
    if (opModeIsActive()) {
      RightFront.setZeroPowerBehavior(DcMotor.ZeroPowerBehavior.BRAKE);
      LeftFront.setZeroPowerBehavior(DcMotor.ZeroPowerBehavior.BRAKE);
      RightBack.setZeroPowerBehavior(DcMotor.ZeroPowerBehavior.BRAKE);
      LeftBack.setZeroPowerBehavior(DcMotor.ZeroPowerBehavior.BRAKE);
      lift.setZeroPowerBehavior(DcMotor.ZeroPowerBehavior.BRAKE);
      Camera2();
      if (CapStoneLocation == false) {
        // Bottom
        for (int count3 = 0; count3 < 100; count3++) {
          lift.setPower(-1);
          telemetry.update();
        }
      } else if (CapStoneLocation > 300) {
        // Top
        for (int count4 = 0; count4 < 600; count4++) {
          lift.setPower(-1);
          telemetry.update();
        }
      } else {
        if (CapStoneLocation < 300) {
          // Middle
          for (int count5 = 0; count5 < 323; count5++) {
            lift.setPower(-1);
            telemetry.update();
          }
        }
      }
      lift.setPower(0);
      for (int count6 = 0; count6 < 20; count6++) {
        RightBack.setPower(0.8);
        LeftBack.setPower(-0.8);
        RightFront.setPower(0.8);
        LeftFront.setPower(-0.8);
        telemetry.update();
      }
      if (Team == -1) {
        for (int count7 = 0; count7 < 66; count7++) {
          RightFront.setPower(-0.3);
          LeftFront.setPower(-0.4);
          RightBack.setPower(-0.3);
          LeftBack.setPower(-0.4);
          telemetry.update();
        }
      }
      if (Team == 1) {
        for (int count8 = 0; count8 < 70; count8++) {
          RightFront.setPower(0.4);
          LeftFront.setPower(0.3);
          RightBack.setPower(0.4);
          LeftBack.setPower(0.3);
          telemetry.update();
        }
      }
      for (int count9 = 0; count9 < 385; count9++) {
        RightFront.setPower(0.7);
        LeftFront.setPower(-0.7);
        RightBack.setPower(0.7);
        LeftBack.setPower(-0.7);
        telemetry.update();
      }
      RightFront.setPower(0);
      LeftFront.setPower(0);
      RightBack.setPower(0);
      LeftBack.setPower(0);
      sleep(1000);
      claw.setPower(0);
      sleep(1000);
      for (int count10 = 0; count10 < 100; count10++) {
        RightFront.setPower(-0.8);
        LeftFront.setPower(0.8);
        RightBack.setPower(-0.8);
        LeftBack.setPower(0.8);
        telemetry.update();
      }
      RightFront.setPower(0);
      LeftFront.setPower(0);
      RightBack.setPower(0);
      LeftBack.setPower(0);
      claw.setPower(0.5);
      sleep(1000);
      if (CapStoneLocation == false) {
        // Bottom
        for (int count11 = 0; count11 < 130; count11++) {
          lift.setPower(1);
          telemetry.update();
        }
      } else if (CapStoneLocation > 300) {
        // Top
        for (int count12 = 0; count12 < 450; count12++) {
          lift.setPower(1);
          telemetry.update();
        }
      } else {
        if (CapStoneLocation < 300) {
          // Middle
          for (int count13 = 0; count13 < 300; count13++) {
            lift.setPower(1);
            telemetry.update();
          }
        }
      }
      lift.setPower(0);
      for (int count14 = 0; count14 < 250; count14++) {
        RightFront.setPower(-0.6);
        LeftFront.setPower(0.6);
        RightBack.setPower(-0.6);
        LeftBack.setPower(0.6);
        telemetry.update();
      }
      if (Team == -1) {
        for (int count15 = 0; count15 < 70; count15++) {
          RightFront.setPower(0.4);
          LeftFront.setPower(0.3);
          RightBack.setPower(0.4);
          LeftBack.setPower(0.3);
          telemetry.update();
        }
      }
      if (Team == 1) {
        for (int count16 = 0; count16 < 70; count16++) {
          RightFront.setPower(-0.3);
          LeftFront.setPower(-0.4);
          RightBack.setPower(-0.3);
          LeftBack.setPower(-0.4);
          telemetry.update();
        }
      }
      for (int count17 = 0; count17 < 170; count17++) {
        RightFront.setPower(-0.6);
        LeftFront.setPower(0.6);
        RightBack.setPower(-0.6);
        LeftBack.setPower(0.6);
        telemetry.update();
      }
      if (Team == -1) {
        for (int count18 = 0; count18 < 380; count18++) {
          RightFront.setPower(-0.3);
          LeftFront.setPower(-0.4);
          RightBack.setPower(-0.3);
          LeftBack.setPower(-0.4);
          telemetry.update();
        }
      }
      if (Team == 1) {
        for (int count19 = 0; count19 < 390; count19++) {
          RightFront.setPower(0.4);
          LeftFront.setPower(0.3);
          RightBack.setPower(0.4);
          LeftBack.setPower(0.3);
          telemetry.update();
        }
      }
      RightFront.setPower(0);
      LeftFront.setPower(0);
      RightBack.setPower(0);
      LeftBack.setPower(0);
      if (Team == -1) {
        for (int count20 = 0; count20 < 445; count20++) {
          RightFront.setPower(-0.5);
          LeftFront.setPower(0.61);
          RightBack.setPower(-0.5);
          LeftBack.setPower(0.61);
          telemetry.update();
        }
      }
      if (Team == 1) {
        for (int count21 = 0; count21 < 455; count21++) {
          RightFront.setPower(-0.58);
          LeftFront.setPower(0.5);
          RightBack.setPower(-0.58);
          LeftBack.setPower(0.5);
          telemetry.update();
        }
      }
      RightFront.setPower(-0.05);
      LeftFront.setPower(0.05);
      RightBack.setPower(-0.05);
      LeftBack.setPower(0.05);
      for (int count22 = 0; count22 < 2200; count22++) {
        telemetry.update();
        spin.setPower(-0.7 * Team);
      }
      RightFront.setPower(0);
      LeftFront.setPower(0);
      RightBack.setPower(0);
      LeftBack.setPower(0);
      spin.setPower(0);
      for (int count23 = 0; count23 < 50; count23++) {
        RightFront.setPower(0.8);
        LeftFront.setPower(-0.8);
        RightBack.setPower(0.8);
        LeftBack.setPower(-0.8);
        telemetry.update();
      }
      if (Team == -1) {
        for (int count24 = 0; count24 < 300; count24++) {
          RightFront.setPower(0.4);
          LeftFront.setPower(0.3);
          RightBack.setPower(0.4);
          LeftBack.setPower(0.3);
          telemetry.update();
        }
      }
      if (Team == 1) {
        for (int count25 = 0; count25 < 400; count25++) {
          RightFront.setPower(-0.3);
          LeftFront.setPower(-0.4);
          RightBack.setPower(-0.3);
          LeftBack.setPower(-0.4);
          telemetry.update();
        }
      }
      RightFront.setPower(0);
      LeftFront.setPower(0);
      RightBack.setPower(0);
      LeftBack.setPower(0);
      for (int count26 = 0; count26 < 250; count26++) {
        RightFront.setPower(0.8);
        LeftFront.setPower(-0.8);
        RightBack.setPower(0.8);
        LeftBack.setPower(-0.8);
        telemetry.update();
      }
      RightFront.setPower(0);
      LeftFront.setPower(0);
      RightBack.setPower(0);
      LeftBack.setPower(0);
    }

    vuforiaFreightFrenzy.close();
    tfodFreightFrenzy.close();
  }
}

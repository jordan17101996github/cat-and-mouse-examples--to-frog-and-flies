# testin-demo

This is the 1st sound sample code of the animation

![screen grab](https://github.com/jordan17101996github/testin-demo/blob/master/Screen%20Shot%202017-11-30%20at%2014.41.01.png)

I am attempting to make a parody code of this by editing the Cat and Mouse into Frog and fly. It was too difficult.

![Screen shot](https://github.com/jordan17101996github/testin-demo/blob/master/Screen%20Shot%202017-11-22%20at%2012.22.39.png)

First code:
Cat and Mouse
import game2dai.entities.*;
import game2dai.entityshapes.ps.*;
import game2dai.maths.*;
import game2dai.*;
import game2dai.entityshapes.*;
import game2dai.fsm.*;
import game2dai.steering.*;
import game2dai.utils.*;
import game2dai.graph.*;

final int SAFE_HERE    = 101;
final int AFTER_YOU    = 102;
final int DIE         = 103;
final int NEXT_MOUSE  = 104;

final int NBR_MICE    = 3;

final float FROG_WANDER_SPEED  = 70;  
final float FROG_CHASE_SPEED    = 160;  
final float FROG_SEEK_SPEED    = 110;  

final float FLY_WANDER_SPEED  = 50;  
final float FLY_EVADE_SPEED  = 90;

// 4 states for the cat
FrogWanderState frogWanderState = new CatWanderState();
ChaseMouseState chaseMouseState = new ChaseMouseState();
SeekMouseState seekMouseState = new SeekMouseState();
KillMouseState killMouseState = new KillMouseState();
// 3 states for the mouse
MouseGlobalState mouseGlobalState = new MouseGlobalState();
MouseWanderState mouseWanderState = new MouseWanderState();
EvadeCatState evadeCatState = new EvadeCatState();

World world;
StopWatch sw;
Cat cat;
Mouse[] mice = new Mouse[NBR_MICE];

float killDistSq;
String legend;
float legendOffsetX;

// Cat and Mouse Simulator
public void setup() {
  size(600, 600);
  world = new World(width, height);
  textSize(12);
  legend = "HINTS [1]-Heading  [2]-Velocity  [3]-Wander  [4]-View  [5]-Obstacle detection  [0]-All off";
  legendOffsetX = (width - textWidth(legend))/2;
  createBarrels();
  createCat();
  createMice();

  reset();
  killDistSq = (float) (cat.colRadius() + mice[0].colRadius());
  killDistSq *= (1.2f * killDistSq);

  sw = new StopWatch();
}

public void draw() {
  double elapsedTime = sw.getElapsedTime();
  world.update(elapsedTime);
  background(200);
  fill(0);
  text(legend, legendOffsetX, height - 6);
  world.draw(elapsedTime);
}

public void keyTyped() {
  int selectedHint = -1;
  switch(key) {
  case '1':
    selectedHint = Hints.HINT_HEADING;
    break;
  case '2':
    selectedHint = Hints.HINT_VELOCITY;
    break;
  case '3':
    selectedHint = Hints.HINT_WANDER;
    break;
  case '4':
    selectedHint = Hints.HINT_VIEW;
    break;
  case '5':
    selectedHint = Hints.HINT_OBS_AVOID;
    break;
  case '0':
    selectedHint = 0;
    break;
  }
  if (selectedHint == 0) { // remove all hints
    cat.renderer().showHints(selectedHint);
    for (Mouse mouse : mice)
      mouse.renderer().showHints(selectedHint);
  }
  else if (selectedHint > 0) { // toggle hint selected
    boolean hintOn = (cat.renderer().getHints() & selectedHint) == selectedHint;
    if (hintOn) {
      cat.renderer().removeHints(selectedHint);
      for (Mouse mouse : mice)
        mouse.renderer().removeHints(selectedHint);
    }
    else {
      cat.renderer().addHints(selectedHint);
      for (Mouse mouse : mice)
        mouse.renderer().addHints(selectedHint);
    }
  }
}

public void createBarrels() {
  int innerCol = color(233, 160, 92);
  int outerCol = color(127, 55, 7);
  float border = 3.2f;
  ObstaclePic view;
  view = new ObstaclePic(this, innerCol, outerCol, border);

  Obstacle[] barrels = Obstacle.makeFromXML(this, "cm_obstacles.xml");
  for (Obstacle barrel : barrels) {
    barrel.renderer(view);
    world.add(barrel);
  }
}

public void createMice() {
  for (int i = 0; i < mice.length; i++) {
    Mouse mouse = new Mouse(Vector2D.ZERO, // position
    10, // collision radius
    Vector2D.ZERO, // velocity
    MOUSE_WANDER_SPEED, // maximum speed
    Vector2D.random(null), // heading
    1, // mass
    2.5f, // turning rate
    2000 // max force
    ); 
    Domain mouseDomain = new Domain(-10, -10, width+10, height+10);
    mouse.worldDomain(mouseDomain, SBF.WRAP);
    mouse.viewFactors(100, 0.9f*PApplet.TWO_PI);
    mouse.renderer(new MousePic(this, 20));
    mouse.AP().wanderFactors(null, 30, 30);
    mouse.AP().obstacleAvoidDetectBoxLength(30);
    mouse.FSM().setGlobalState(mouseGlobalState);
    mouse.FSM().changeState(mouseWanderState);
    world.add(mouse);
    mice[i] = mouse;
  }
}

public void createCat() {
  cat = new Cat(Vector2D.ZERO, // position
  20, // collision radius
  Vector2D.ZERO, // velocity
  CAT_WANDER_SPEED, // maximum speed
  Vector2D.random(null), // heading
  1.5, // mass
  2.5f, // turning rate
  2500 // max force
  ); 
  Domain catDomain = new Domain(12, 12, width-12, height-12);
  cat.worldDomain(catDomain, SBF.REBOUND);
  cat.viewFactors(260, PApplet.TWO_PI/7);
  cat.renderer(new CatPic(this, 40));
  cat.AP().wanderFactors(100, 40, 20);
  cat.AP().obstacleAvoidDetectBoxLength(40);
  world.add(cat);
}

public void reset() {
  // Remove all pending births and deaths
  //    world.cancelBirthsAndDeaths();
  Vector2D pos;
  // Starting position for the cat
  pos = Vector2D.random(null).mult(50);
  cat.moveTo(pos.x + 150, pos.y + 150);
  cat.velocity(0, 0);
  cat.AP().obstacleAvoidOn();
  cat.FSM().changeState(catWanderState);
  cat.miceKilled = 0;
  world.add(cat);
  // Starting positions for mice
  int[] dx = { 
    150, 450, 450
  };
  int[] dy = { 
    450, 450, 150
  };
  for (int i = 0; i < mice.length; i++) {
    pos = Vector2D.random(null).mult(50);
    mice[i].alive = true;
    mice[i].moveTo(pos.x + dx[i], pos.y + dy[i]);
    mice[i].velocity(0, 0);
    mice[i].AP().obstacleAvoidOn();
    mice[i].FSM().changeState(mouseWanderState);
    world.add(mice[i]);
  }
}
Cat:
public class Cat extends Vehicle {

  public Mouse chasing = null;
  public Vector2D lastKnownPos = new Vector2D();
  public int miceKilled = 0;

  public Cat(Vector2D position, double radius, Vector2D velocity, 
  double max_speed, Vector2D heading, double mass, 
  double max_turn_rate, double max_force) {
    super(position, radius, velocity, max_speed, heading, mass, max_turn_rate, 
    max_force);
    addFSM();
  }

  public void lookForMouse() {
    chasing = null;
    for (int i = 0; i < mice.length; i++) {
      if (mice[i].alive &&  canSee(world, mice[i].pos())) {
        chasing = mice[i];
        lastKnownPos.set(chasing.pos());
      }
    }
  }

  public void adjustLastKnownPosition() {
    // Adjust x position
    if (lastKnownPos.x < wd.lowX)
      lastKnownPos.x = wd.lowX;
    else if (lastKnownPos.x > wd.highX)
      lastKnownPos.x = wd.highX;
    // Adjust y position
    if (lastKnownPos.y < wd.lowY)
      lastKnownPos.y = wd.lowY;
    else if (lastKnownPos.y > wd.highY)
      lastKnownPos.y = wd.highY;
  }
} // End of Cat class



public class CatPic extends PicturePS {

  int head, eye, whiskers;
  float size;

  public CatPic(PApplet app, float size, int body, int eye, int whiskers) {
    super(app);
    this.size = size;
    this.head = body;
    this.eye = eye;
    this.whiskers = whiskers;
  }

  public CatPic(PApplet app, float size) {
    this(app, size, color(255, 169, 19), color(100, 100, 200), color(0));
  }


  public void draw(BaseEntity user, float posX, float posY, float velX, 
  float velY, float headX, float headY, float etime) {

    // Draw and hints that are specified and relevant
    if (hints != 0) {
      Hints.hintFlags = hints;
      Hints.draw(app, user, velX, velY, headX, headY);
    }
    // Determine the angle the tank is heading
    float angle = PApplet.atan2(headY, headX);

    // Prepare to draw the entity    
    pushStyle();
    pushMatrix();
    translate(posX, posY);
    rotate(angle);

    // Draw the entity  
    ellipseMode(PApplet.CENTER);
    stroke(whiskers);
    strokeWeight(1);
    line(0.3f*size, -0.35f*size, 0.4f*size, 0.35f*size);
    line(0.4f*size, -0.35f*size, 0.3f*size, 0.35f*size);

    stroke(whiskers);
    strokeWeight(0.5f);
    fill(head);

    arc(0.05f*size, 0, 0.5f*size, 1.2f*size, PApplet.HALF_PI - 0.1f, 3* PApplet.HALF_PI + 0.1f, PApplet.CHORD);
    ellipse(0, 0, 0.7f*size, 0.7f*size);

    fill(whiskers);
    ellipse(0.35f*size, 0, 0.14f*size, 0.2f*size);

    fill(eye);
    ellipse(0.12f*size, 0.18f*size, 0.12f*size, 0.22f*size);
    ellipse(0.12f*size, -0.18f*size, 0.12f*size, 0.22f*size);

    // Finished drawing
    popMatrix();
    popStyle();
  }
} // End of CatPic class
Mouse:
public class Mouse extends Vehicle {

  boolean alive = true;

  public Mouse(Vector2D position, double radius, Vector2D velocity, 
  double max_speed, Vector2D heading, double mass, 
  double max_turn_rate, double max_force) {
    super(position, radius, velocity, max_speed, heading, mass, max_turn_rate, 
    max_force);
    addFSM();
  }
} // End of Mouse class



public class MousePic extends PicturePS {

  int head, eye, whiskers;
  float size;

  public MousePic(PApplet app, float size, int body, int eye, int whiskers) {
    super(app);
    this.size = size;
    this.head = body;
    this.eye = eye;
    this.whiskers = whiskers;
  }

  public MousePic(PApplet app, float size) {
    this(app, size, color(160), color(255, 200, 200), color(0));
  }


  public void draw(BaseEntity user, float posX, float posY, float velX, 
  float velY, float headX, float headY, float etime) {

    // Draw and hints that are specified and relevant
    if (hints != 0) {
      Hints.hintFlags = hints;
      Hints.draw(app, user, velX, velY, headX, headY);
    }
    // Determine the angle the tank is heading
    float angle = PApplet.atan2(headY, headX);

    // Prepare to draw the entity    
    pushStyle();
    ellipseMode(PApplet.CENTER);
    pushMatrix();
    translate(posX, posY);
    rotate(angle);

    // Draw the entity  
    stroke(whiskers);
    strokeWeight(1);
    line(0.4f*size, -0.5f*size, 0.6f*size, 0.5f*size);
    line(0.6f*size, -0.5f*size, 0.4f*size, 0.5f*size);

    strokeWeight(0.5f);
    fill(head);
    arc(0.15f*size, 0, 0.3f*size, 1.2f*size, PApplet.HALF_PI - 0.4f, 3* PApplet.HALF_PI + 0.4f, PApplet.CHORD);
    arc(0, 0, 0.7f*size, 0.7f*size, PApplet.HALF_PI, 3* PApplet.HALF_PI);
    arc(0, 0, 1.1f*size, 0.7f*size, 3* PApplet.HALF_PI, PApplet.TWO_PI);
    arc(0, 0, 1.1f*size, 0.7f*size, 0, PApplet.HALF_PI);

    fill(whiskers);
    ellipse(0.55f*size, 0, 0.2f*size, 0.2f*size);

    fill(eye);
    ellipse(0.12f*size, 0.15f*size, 0.2f*size, 0.22f*size);
    ellipse(0.12f*size, -0.15f*size, 0.2f*size, 0.22f*size);


    // Finished drawing
    popMatrix();
    popStyle();
  }
} // End of MousePic class
States:
public class CatWanderState extends State {

  public void enter(BaseEntity user) {
    Cat c = (Cat)user;
    c.maxSpeed(CAT_WANDER_SPEED);
    c.AP().wanderOn();
  }

  public void execute(BaseEntity user, double deltaTime, World world) {
    Cat c = (Cat)user;
    c.lookForMouse();
    if (c.chasing != null)
      c.FSM().changeState(chaseMouseState);
  }

  public void exit(BaseEntity user) {
    Cat c = (Cat)user;
    c.AP().wanderOff();
  }

  public boolean onMessage(BaseEntity user, Telegram tgram) {
    return false;
  }
} // End of CatWanderState class


public class ChaseMouseState extends State {

  public void enter(BaseEntity user) {
    Cat c = (Cat)user;
    c.maxSpeed(CAT_CHASE_SPEED);
    Dispatcher.dispatch(500, c.ID(), c.chasing.ID(), AFTER_YOU);
    c.AP().pursuitOn(c.chasing);
  }

  public void execute(BaseEntity user, double deltaTime, World world) {
    Cat c = (Cat)user;
    if (Vector2D.distSq(c.pos(), c.chasing.pos()) < killDistSq) {
      c.FSM().changeState(killMouseState);
    }
    else if (!c.canSee(world, c.chasing.pos())) {
      c.FSM().changeState(seekMouseState);
    }
  }

  public void exit(BaseEntity user) {
    Cat c = (Cat)user;
    c.AP().pursuitOff();
  }

  public boolean onMessage(BaseEntity user, Telegram tgram) {
    return false;
  }
} // End of ChaseMouseState class


public class KillMouseState extends State {

  public void enter(BaseEntity user) {
    Cat c = (Cat)user;
    c.velocity(0, 0);
    c.miceKilled++;
    // Send mouse a message to die
    Dispatcher.dispatch(0, c.ID(), c.chasing.ID(), DIE);
    // Slight pause before back to wander
    Dispatcher.dispatch(2000, c.ID(), c.ID(), NEXT_MOUSE);
  }

  public void execute(BaseEntity user, double deltaTime, World world) {
  }

  public void exit(BaseEntity user) {
  }

  public boolean onMessage(BaseEntity user, Telegram tgram) {
    Cat c = (Cat)user;
    switch(tgram.msg) {
    case NEXT_MOUSE:
      if (c.miceKilled < NBR_MICE)
        c.FSM().changeState(catWanderState);
      else
        reset();
      return true;
    }      
    return false;
  }
} // End of CatWaitState class


public class SeekMouseState extends State {

  public void enter(BaseEntity user) {
    Cat c = (Cat)user;
    c.maxSpeed(CAT_SEEK_SPEED);
    c.adjustLastKnownPosition();
    c.AP().arriveOn(c.lastKnownPos);
  }

  public void execute(BaseEntity user, double deltaTime, World world) {
    Cat c = (Cat)user;
    // Can we see a mouse to chase
    c.lookForMouse();
    if (c.chasing != null) {
      c.FSM().changeState(chaseMouseState);
    }
    else if (c.AP().arriveDistance() < 20) {
      c.FSM().changeState(catWanderState);
    }
  }

  public void exit(BaseEntity user) {
    Cat c = (Cat)user;
    c.AP().arriveOff();
  }

  public boolean onMessage(BaseEntity user, Telegram tgram) {
    return false;
  }
} // End of SeekMouseState class


public class MouseGlobalState extends State {

  public void enter(BaseEntity user) {
  }

  public void execute(BaseEntity user, double deltaTime, World world) {
  }

  public void exit(BaseEntity user) {
  }

  public boolean onMessage(BaseEntity user, Telegram tgram) {
    Mouse m = (Mouse)user;
    switch(tgram.msg) {
    case DIE:
      m.alive = false;
      m.velocity(0, 0);
      m.AP().allOff();
      m.die(world, 0.5);
      return true;
    }      
    return false;
  }
} // End of MouseGlobalState class


public class MouseWanderState extends State {

  public void enter(BaseEntity user) {
    Mouse m = (Mouse)user;
    m.maxSpeed(MOUSE_WANDER_SPEED);
    m.AP().wanderOn();
  }

  public void execute(BaseEntity user, double deltaTime, World world) {
    Mouse m = (Mouse)user;
    if (m.canSee(world, cat.pos()))
      m.FSM().changeState(evadeCatState);
  }

  public void exit(BaseEntity user) {
    Mouse m = (Mouse)user;
    m.AP().wanderOff();
  }

  public boolean onMessage(BaseEntity user, Telegram tgram) {
    Mouse m = (Mouse)user;
    switch(tgram.msg) {
    case AFTER_YOU:
      m.FSM().changeState(evadeCatState);
      return true;
    }      
    return false;
  }
} // End of MouseWanderState class


public class EvadeCatState extends State {

  public void enter(BaseEntity user) {
    Mouse m = (Mouse)user;
    m.maxSpeed(MOUSE_EVADE_SPEED);
    if (Vector2D.dist(m.pos(), cat.pos()) > 100)
      m.AP().hideOn(cat);
    else
      m.AP().evadeOn(cat);
  }

  public void execute(BaseEntity user, double deltaTime, World world) {
    Mouse m = (Mouse)user;
    if (!cat.canSee(world, m.pos()))
      Dispatcher.dispatch(1000, m.ID(), m.ID(), SAFE_HERE);
  }

  public void exit(BaseEntity user) {
    Mouse m = (Mouse)user;
    m.AP().hideOff();
    m.AP().evadeOff();
  }

  public boolean onMessage(BaseEntity user, Telegram tgram) {
    Mouse m = (Mouse)user;
    switch(tgram.msg) {
    case SAFE_HERE:
      m.FSM().changeState(mouseWanderState);
      return true;
    }      
    return false;
  }
} // End of MouseWanderState class

# Getting Started with Unity - Code Snippet

```
Assets
    └── prefabs
    └── scenes
    └── scripts
    └── sounds
    └── sprites
```

## 1. Decide Player Object
## 2. Game scene
## 3. UI, Audio, Particles and Assets
## 4. Code Refactor

## 1. Decide Player Object

https://github.com/michidk/Unity-Script-Collection#character-controller

1. Player Movement


```C#
// #### PlayerController.cs  #####

private Rigidbody playerRb;

// Start()
playerRb = GetComponent<Rigidbody>();


// FixedUpdate()

Vector3 direction = new Vector3(Input.GetAxisRaw("Horizontal"), 0 , Input.GetAxisRaw("Vertical"));
rb.AddForce(direction*speed);
rigidBody.AddForce(0, 0, speed * Time.deltaTime, ForceMode.VelocityChange);

// ######### Move Position smoothly
rigidbody.MovePosition(transform.position + direction * movementSpeed * Time.fixedDeltaTime);


// #### To teletransport the GameObject, without considering physics nor colliders use transform #####
transform.Translate(Vector3.right * horizontalInput * Time.deltaTime * sideSpeed);
transform.Rotate(Vector3.up, spinSpeed * Time.deltaTime);
```


2. Collision
Add after setting up Scene and enemys
```C#
void OnTriggerEnter(Collider other){
	if (other.gameObject.CompareTag ("PowerPickUp")) {
		other.gameObject.SetActive (false);
        // GameManager.pickups("PowerPickUp");
        Destroy(other.gameObject);
	}
}


```

## 2. Game scene
**1. Game Type (Level Based or Infinite)**  
**2. Camera to follow player**  
**3. Background (static, repeating)**  
**4. Obstacle PowerUps and Enemies (static or spawing)**  


- Camera to follow player
```C#
public GameObject player;

	private Vector3 offset;

	// Use this for initialization
	void Start () {
		offset = transform.position - player.transform.position;
	}
	
	// Update is called once per frame
	void LateUpdate () {
		transform.position = player.transform.position + offset;
	}
```

- Repeating Background or repeating movement
```C#
    
    private Vector3 startPos;
    private float repeatWidth;

    private void Start()
    {
        // Establish the default starting position  
        startPos = transform.position;  
        // Set repeat width to half of the background  
        repeatWidth = GetComponent<BoxCollider>().size.x / 2; 
        
    }

    private void Update()
    {
        // If background moves left by its repeat width, move it back to start position
        if (transform.position.x < startPos.x - repeatWidth)
        {
            transform.position = startPos;
        }
    }
```

- Obstacle PowerUps and Enemies Spawning
```C#
    public GameObject[] objectPrefabs;
    public float spawnDelay = 2;
    public float spawnInterval = 1.5f;
    public float spawnRangeX = 20f;
    public float spawnRangeZ = 15.0f;

    private PlayerControllerX playerControllerScript;

    // Start is called before the first frame update
    void Start()
    {
        InvokeRepeating("SpawnObjects", spawnDelay, spawnInterval);
        playerControllerScript = GameObject.Find("Player").GetComponent<PlayerControllerX>();
    }

    // Spawn obstacles
    void SpawnObjects ()
    {
        // Set random spawn location and random object index
        Vector3 spawnLocation = new Vector3(Random.Range(-spawnRangeX, spawnRangeX), 0, spawnRangeZ);
        int index = Random.Range(0, objectPrefabs.Length);

        // If game is still active, spawn new object
        if (!playerControllerScript.gameOver)
        {
            Instantiate(objectPrefabs[index], spawnLocation, objectPrefabs[index].transform.rotation);
        }

    }
```

## 3. UI, Audio, Particles and Assets


```C#
public ParticleSystem explosionParticle;
public AudioClip explodeSound;


explosionParticle.Play();
playerAudio.PlayOneShot(explodeSound, 1.0f);

```


## 4. Code Refactor


1. Script reference - (Communicate between classes)
```C#
private PlayerControllerX playerControllerScript;

// initialize - Start()
playerControllerScript = GameObject.Find("Player").GetComponent<PlayerControllerX>();

// use
playerControllerScript.isGameOver;
```

2. Events - (Communicate between classes) **RECOMMEDED*

```C#
// ### PlayerController.cs ###

// Define event
public delegate void PlayerDelegate ();
public static event PlayerDelegate onPlayerScored;

// Fire Event, need check for null - if there are no subscriber it will return error
// this will call all the methods that are subscribed to PlayerController.onPlayerScored
if (onPlayerScored != null) {
    onPlayerScored();
}


// ### GameManager.cs ###
// subscribe to the event and call required function
PlayerController.onPlayerScored += increasePlayerScore;

void increasePlayerScore() {
    score++;
    scoreText.text = score.ToString ();
}

// Note: unSubscribe to the event whenver it is no longer required (on object destoryed or OnDisable() ) 
PlayerController.onPlayerScored += increasePlayerScore;

```


3. Code Structure
```C#

// [SerializeField] allows value to be assined from unity editor
 [SerializeField] private float speed;

// Define script to be assigned to
[RequireComponent(typeof(Rigidbody2D))]
public class PlayerController : MonoBehaviour

// Use Quaternion for rotations
Quaternion downRotation = Quaternion.Euler (0, 0, -90);
Quaternion forwardRotation = Quaternion.Euler (0, 0, 35);

// Use event delegates for communication

 ```


 4. Some Hints
 
 - Use enum to manage UI state and mapped through them, #MenuManager.cs

 ```C#

    public GameObject startPageUI;
	public GameObject gameOverPageUI;
	public GameObject countDownPageUI;
	public Text highScoreText;

	enum PageState {
	    None,
	    Start,
        Pause,
	    GameOver,
        Countdown
	}

    void SetPageState(PageState state){
		switch (state) {
		case PageState.None:
			startPage.SetActive (false);
			gameOverPage.SetActive (false);
			countDownPage.SetActive (false);
			break;
		case PageState.Countdown:
			startPage.SetActive (false);
			gameOverPage.SetActive (false);
			countDownPage.SetActive (true);
			break;
		case PageState.Start:
			startPage.SetActive (true);
			gameOverPage.SetActive (false);
			countDownPage.SetActive (false);
			break;
		case PageState.GameOver:
			startPage.SetActive (false);
			gameOverPage.SetActive (true);
			countDownPage.SetActive (false);
			break;
		}

        // Methods
        public void StartGame() {
		    // activate when play button
		    OnGameStarted ();
        }

        // .........

	}

```


- Loading Scence, Level Complete, restart and Quit

```C#

public GameObject levelCompleteUI;

PlayerController.onPlayerDied += Restart;

void Restart()
    {
        SceneManager.LoadScene(SceneManager.GetActiveScene().name);
    }
    
public void OnLevelComplete()
    {
        levelCompleteUI.SetActive(true);
        SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex + 1);
    }

    // To Quit Application
    if (UNITY_EDITOR) {
        EditorApplication.ExitPlaymode();
    }
    else {
        Application.Quit(); 
    }
    
    
}

```


### Further Unity Samples
https://github.com/michidk/Unity-Script-Collection 
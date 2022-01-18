# README.md

# Madcamp_week3

## Project Name: Mokoko’s Adventure

- Project Member
    
    Jung Eun Ho
    
    Choi Dae gun
    
- Project Toolkit / Language
    
    Unity / C
    

- Game component
    1. Scene
        
        시작화면인 Main scene과 던전화면인 Dungeon1 scene으로 구성됨.
        
    2. Monster
        
        몬스터들은 플레이어의 위치를 확인 & 거리를 비교해서 chase Length보다 짧다면 따라오도록 구현. Collider들이 overlap되면 데미지를 받고 exp를 플레이어에게 줌.
        
        ```csharp
        using UnityEngine;
        
        public class Enemy : Mover
        {
            // Experience
            public int xpValue = 1;
        
            // Logic
            public float triggerLength = 1;
            public float chaseLength = 5;
            private bool chasing;
            private bool collidingWithPlayer;
            private Transform playerTransform;
            private Vector3 startingPosition;
        
            // Hitbox
            private BoxCollider2D hitbox;
        
            protected override void Start()
            {
                base.Start();
                playerTransform = GameManager.instance.player.transform;
                startingPosition = transform.position;
                xSpeed = 0.75f;
                ySpeed = 0.5f;
                hitbox = transform.GetChild(0).GetComponent<BoxCollider2D>();
            }
        
            protected override void FixedUpdate()
            {
                base.FixedUpdate();
                // Is the player in range?
                if (Vector3.Distance(playerTransform.position, startingPosition) < chaseLength)
                {
                    chasing = Vector3.Distance(playerTransform.position, startingPosition) < triggerLength || chasing;
        
                    if (chasing)
                    {
                        if (!collidingWithPlayer)
                        {
                            UpdateMotor((playerTransform.position - transform.position).normalized);
                        }
                    }
                    else
                    {
                        UpdateMotor(startingPosition - transform.position);
                        chasing = false;
                    }
        
                }
                else
                {
                    UpdateMotor(startingPosition - transform.position);
                    chasing = false;
                }
        
                // Check for overlap
                collidingWithPlayer = false;
                boxCollider.OverlapCollider(filter, hits);
        
                for (int i = 0; i < hits.Length; i++)
                {
                    if (hits[i] == null)
                        continue;
        
                    if (hits[i].tag == "Fighter" && hits[i].name == "Player")
                    {
                        collidingWithPlayer = true;
                    }
        
                    hits[i] = null;
                }
            }
        
            protected override void Death()
            {
                Destroy(gameObject);
                GameManager.instance.experience += xpValue;
                GameManager.instance.ShowText("+" + xpValue.ToString() + "xp", 30, Color.magenta, transform.position, Vector3.up * 25, 1.0f);
            }
        }
        ```
        
    3. Player
        
        체력을 가지는 클래스인 Fighter를 상속하면서 움직임을 담당하는 클래스인 mover를 플레이어가 상속받는다. 플레이어에는 공격력, HP, MP, 경험치, 레벨 시스템을 가지고 있으며 레벨이 오를 때마다 HP와 공격력이 늘어나며 HP가 full이 되도록 구현했다.
        
        
        <img width="645" alt="스크린샷 2022-01-18 오후 8 46 47" src="https://user-images.githubusercontent.com/32477937/149935777-e2946b33-24e9-4656-8dc4-23170b51f1db.png">
        
        
        ![levelup](https://user-images.githubusercontent.com/32477937/149936393-8bd9b8b1-c994-4b25-baf1-3fa4324b4bbe.gif)

        ```csharp
        using System.Collections;
        using System.Collections.Generic;
        using UnityEngine;
        using UnityEngine.UI;
        public class Player : Mover
        {
            private Text levelText;
            private Animator animator;
            public bool attacked = false;
            public Image nowHpbar;
            public int atkdmg;
            public int Mylevel;
        
            public int reference_exp = 20;
            public int next_level_exp = 20;
        
            protected override void Start(){
                base.Start();
                Mylevel = 1;
                atkdmg = 30;
                hitpoint = 100;
                maxHitpoint = 100;
                animator = GetComponent<Animator>();
                levelText = GameObject.Find("Levelnum").GetComponent<Text>();
                DontDestroyOnLoad(gameObject);
            }
        
            protected override void FixedUpdate()
            {
                base.FixedUpdate();
                nowHpbar.fillAmount = (float)hitpoint / (float)maxHitpoint;
                float x = Input.GetAxisRaw("Horizontal");
                float y = Input.GetAxisRaw("Vertical");
        
                if(Input.GetKeyDown(KeyCode.B)){
                    Debug.Log("player attakced");
                    animator.SetTrigger("attack");
                    attacked = true;
                }
                if(x!=0 || y != 0){
                    animator.SetBool("walk", true);
                }
                else{
                    animator.SetBool("walk", false);
                }
        
                if(next_level_exp <= 0){
                    Levelup();
                }
                UpdateMotor(new Vector3(x,y,0));
            }
        
            void Levelup(){
                Mylevel++;
                atkdmg += 20;
                next_level_exp = reference_exp * Mylevel;
                levelText.text = Mylevel.ToString();
            }
        
            protected override void UpdateMotor(Vector3 input)
            {
                // Reset moveDelta
                moveDelta = new Vector3(input.x * xSpeed, input.y * ySpeed, 0);
        
                // Swap sprite direction, wether you're going right or left
                if (moveDelta.x > 0)
                    transform.localScale = new Vector3(-1, 1, 1);
                else if (moveDelta.x < 0)
                    transform.localScale = Vector3.one;
        
                // Add push vector, if any
                moveDelta += pushDirection;
        
                // Reduce push force every frame, based off recovery speed
                pushDirection = Vector3.Lerp(pushDirection, Vector3.zero, pushRecoverySpeed);
        
                // Make sure we can move in this direction, by casting a box there first, if the box returns null, we're fine to move
                hit = Physics2D.BoxCast(transform.position, boxCollider.size, 0, new Vector2(0, moveDelta.y), Mathf.Abs(moveDelta.y * Time.deltaTime), LayerMask.GetMask("Actor", "Blocking"));
                if (hit.collider == null)
                {
                    //Make this move
                    transform.Translate(0, moveDelta.y * Time.deltaTime, 0);
                }
        
                // Make sure we can move in this direction, by casting a box there first, if the box returns null, we're fine to move
        
                hit = Physics2D.BoxCast(transform.position, boxCollider.size, 0, new Vector2(moveDelta.x, 0), Mathf.Abs(moveDelta.x * Time.deltaTime), LayerMask.GetMask("Actor", "Blocking"));
                if (hit.collider == null)
                {
                    //Make this move
                    transform.Translate(moveDelta.x * Time.deltaTime, 0, 0);
                }
            }
        }
        ```
        
1. Boss
    보스는 두 개의 스크립트로 구성. Boss & BossAI
    
    Boss.cs에서 전반적인 능력치를 다룬다. BossAI.cs에서는 보스의 공격 알고리즘이 구현되어 있다. fieldofvision을 설정해서 특정 거리로 들어오게 되면 보스가 플레이어를 바라보며, 공격 범위 내로 들어오면 공격모션을 취한다. 그 외의 상황에서는 move함수를 통해 플레이어를 향해 이동한다. 보스의 Hp가 절반 이하로 내려가면, 난수를 통해 여러 스킬들을 사용한다. 이 스킬들의 데미지는 매우 높게 설정해 클리어하기 어렵게 설정했다.
    
    <img width="1136" alt="스크린샷 2022-01-18 오후 8 47 28" src="https://user-images.githubusercontent.com/32477937/149935974-189e9dab-0765-4910-a6a6-5cfba958f1ed.png">

       
    ```csharp
    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine;
    using UnityEngine.UI;
    
    public class Boss : MonoBehaviour
    {
        public GameObject prfHpBar;
        public GameObject canvas;
    
        RectTransform hpBar;
        public string enemyName;
        public int maxHp;
        public int nowHp;
        public int atkDmg;
        public float atkSpeed;
        public int exp;
    
        public float moveSpeed;
        public float atkRange;
        public float fieldOfVision;
        public bool attack_bool = false;
        private void Setenemystatus(string _enemyName, int _maxHp, int _atkDmg, float _atkSpeed, float _moveSpeed, float _atkRange, float _fieldOfVision, int _exp){
            enemyName = _enemyName;
            maxHp = _maxHp;
            nowHp = _maxHp;
            atkDmg = _atkDmg;
            atkSpeed = _atkSpeed;
            exp = _exp;
            moveSpeed = _moveSpeed;
            atkRange = _atkRange;
            fieldOfVision = _fieldOfVision;
        }
    
        public Player character;
        public Weapon sword;
        Image nowHpbar;
        public float height = 0.1f;
        public Animator bossAnimator;
    
        // Start is called before the first frame update
        void Start()
        {
            character = GameObject.Find("Player").GetComponent<Player>();
            sword = GameObject.Find("Sword2").GetComponent<Weapon>();
            canvas = GameObject.Find("FloatingTextManager");
            hpBar = Instantiate(prfHpBar, canvas.transform).GetComponent<RectTransform>();
            if(name.Equals("Dragon")){
                Setenemystatus("Dragon", 100, 10, 1.5f, 2, 1.5f, 7f, 20);
            }
            if(name.Equals("Boss")){
                Setenemystatus("Boss", 500, 10, 3.0f, 1.5f, 0.5f, 7.0f, 300);
            }
            nowHpbar = hpBar.transform.GetChild(0).GetComponent<Image>();
        }
    
        // Update is called once per frame
        void Update()
        {
            Vector3 _hpBarPos = Camera.main.WorldToScreenPoint(new Vector3(transform.position.x, transform.position.y + 0.65f, 0));
            hpBar.position = _hpBarPos;
            nowHpbar.fillAmount = (float)nowHp/(float)maxHp;
        }
    
        private void OnTriggerEnter2D(Collider2D col){
            Debug.Log("collision");
            if(character.attacked == true || sword.sword_attacked == true){
                Debug.Log("attacked");
                nowHp -= (character.atkdmg + sword.atkdmg);
                Debug.Log(nowHp);
                character.attacked = false;
                sword.sword_attacked = false;
    
                if(nowHp <= 0){
                    //GetComponent<EnemyAI>().enabled = false;    // 추적 비활성화
                    //GetComponent<Collider2D>().enabled = false; // 충돌체 비활성화
                    Destroy(gameObject);
                    Destroy(hpBar.gameObject);
                    character.next_level_exp -= this.exp;
                }
            }     
            if(attack_bool == true){
                character.hitpoint -= atkDmg;
            }
        }
    
    }
    ```
    
    ```csharp
    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine;
    public class BossAI : MonoBehaviour
    {
        public Transform target;
        float attackDelay;
    
        public Boss boss;
        Animator bossAnimator;
    
        public bool half_hp = false;
    
        public int randomvalue;
        public string skillname;
        void Start()
        {
            boss = GetComponent<Boss>();
            bossAnimator = boss.bossAnimator; 
            target = GameObject.Find("Player").transform;
        }
    
        void choose_skill(){
            boss.attack_bool = true;
            randomvalue = Random.Range(0,5);
            if(boss.nowHp < boss.maxHp/2){
                half_hp = true;
                if(randomvalue == 0){
                    skillname = "attack2";
                    boss.atkDmg = 30;
                }
                else if(randomvalue == 1){
                    skillname = "attack";
                    boss.atkDmg = 10;
                }
                else if(randomvalue == 2)
                {
                    skillname = "counter";
                }
                else if(randomvalue == 3)
                {
                    skillname = "counter1";
                }
                else if(randomvalue == 4)
                {
                    skillname = "counter2";
                }
            }
            else{
                skillname = "attack";
                boss.atkDmg = 10;
            }
        }
        // Update is called once per frame
        void Update()
        {
            attackDelay -= Time.deltaTime;
            if (attackDelay < 0) attackDelay = 0;
            //타겟과 자신의 거리를 확인
            float distance = Vector3.Distance(transform.position, target.position);
            //쿨타임이 0일때 + 시야범위 내에 들어올 때
            if (attackDelay == 0 && distance <= boss.fieldOfVision)
            {
                FaceTarget(); //타겟 바라보기
    
                if (distance <= boss.atkRange)
                {
                    AttackTarget();
                }
                else //공격 애니메이션이 실행중이 아닐 때
                {
                    if (!bossAnimator.GetCurrentAnimatorStateInfo(0).IsName("baltan_skill1"))
                    {
                        MoveToTarget();
                    }
                }
            }
            else //시야범위 밖에 있을 떄는 idle 애니메이션으로 전환.
            {
                bossAnimator.SetBool("moving", false);
            }
            boss.attack_bool = false;
        }
    
            void MoveToTarget()
        {
            float dir1 = target.position.x - transform.position.x;
            dir1 = (dir1 < 0) ? -1 : 1;
            transform.Translate(new Vector2(dir1, 0) * boss.moveSpeed * 1/2 * Time.deltaTime);
            bossAnimator.SetBool("moving", true);
    
            float dir2 = target.position.y - transform.position.y;
            dir2 = dir2 < 0 ?  -1 : 1;
            transform.Translate(new Vector2(0, dir2) * boss.moveSpeed * 1/2 * Time.deltaTime);
            bossAnimator.SetBool("moving", true);
        }
    
        void FaceTarget()
        {
            if (target.position.x - transform.position.x < 0) // 타겟이 왼쪽에 있을 때
            {
                transform.localScale = new Vector3(1, 1, 1);
            }
            else if(target.position.x - transform.position.x > 0) // 타겟이 오른쪽에 있을 때
            {
                transform.localScale = new Vector3(-1, 1, 1);
            }
            // else{
            //     if(target.position.y - transform.position.y < 0){
            //         transform.localScale = new Vector3(1, 1, 1);
            //     }
            //     else{
            //         transform.localScale = new Vector3(1, 1, 1);
            //     }
            // }
        }
    
        void AttackTarget()
        {
            choose_skill();
            bossAnimator.SetTrigger(skillname); // 공격 애니메이션 실행
            target.GetComponent<Player>().hitpoint -= boss.atkDmg;
            attackDelay = boss.atkSpeed; // 딜레이 충전
        }
    }
    ```
    
    1. Display Component
        
        플레이어 캐릭터와 보스몹은 도트를 이용해서 제작했음. 애니메이션들도

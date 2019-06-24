# arc-engine-scene-

arc engine scene组件并不自带三维旋转功能（貌似），搜了一下发现只有缩放scale的，没有旋转的，故自行写了一个鼠标按压移动旋转视图的功能

此功能用到c#的timer控件

//全局变量
System.Drawing.Point pPoint = new System.Drawing.Point()
        {
            X = 0,
            Y = 0
        };
ICamera pCamera = new Camera();
IPoint pPtObs = new ESRI.ArcGIS.Geometry.Point();
IPoint pPtTar = new ESRI.ArcGIS.Geometry.Point();
bool state = false;;//状态
double φdegree;//水平角
double sqrtxy;//xOy平面上的投影长度
double θdegree;//倾角
double r;//视距。即极坐标半径




public Form1()
        {
            InitializeComponent();
            this.timer1 = new System.Windows.Forms.Timer(this.components);
            this.timer1.Tick += new System.EventHandler(this.Timer1_Tick);//绑定事件Timer1_Tick
        }
        
private void Form1_Load(object sender, EventArgs e)
        {
        //////
        
        this.timer1.Enabled = false;
        this.timer1.Interval = 1;
        
        pCamera = axSceneControl1.Camera;
        pPtObs = pCamera.Observer;
        pPtTar = pCamera.Target;
        φdegree = Math.Atan((pPtObs.Y - pPtTar.Y) / (pPtObs.X - pPtTar.X));//反三角函数求极坐标方位角φ
        sqrtxy = Math.Sqrt(Math.Pow(pPtObs.Y - pPtTar.Y, 2) + Math.Pow(pPtObs.X - pPtTar.X, 2));//xOy平面上的投影长度
        θdegree = Math.Atan((pPtObs.Z - pPtTar.Z) / sqrtxy);//反三角函数求极坐标倾角 
        r = Math.Sqrt(Math.Pow(pPtObs.X - pPtTar.X, 2) + Math.Pow(pPtObs.Y - pPtTar.Y, 2) + Math.Pow(pPtObs.Z - pPtTar.Z, 2));//视距。即极坐标半径
        
        //////
        
        }
      
      
private void Timer1_Tick(object sender, EventArgs e)
        {
            if (axSceneControl1.Visible == true && state == true)
            {
                try
                {
                    System.Drawing.Point pSceLoc = axSceneControl1.PointToScreen(this.axSceneControl1.Location);
                    System.Drawing.Point Pt = new System.Drawing.Point();
                    Pt.X = Control.MousePosition.X; 
                    Pt.Y = Control.MousePosition.Y;
                    if (Pt.X < pSceLoc.X | Pt.X > pSceLoc.X + axSceneControl1.Width | Pt.Y <
        pSceLoc.Y | Pt.Y > pSceLoc.Y + axSceneControl1.Height) return;
                    if (Pt.X - pPoint.X > 100)
                    {
                        pPoint = Pt;//Pt为实时鼠标坐标，pPoint为上一鼠标坐标，差大于阈值即赋值
                        return;
                    }
                    double degree = 0.01;
                    
                    double φdegree1 = φdegree + (pPoint.X - Pt.X) * degree;
                    double θdegree1 = θdegree + (Pt.Y - pPoint.Y) * degree;
                    if (θdegree1 > 0.8)
                    {
                        θdegree1 = 0.8;
                    }
                    else if (θdegree1 < 0.5)
                    {
                        θdegree1 = 0.5;
                    }
                    double z = r * Math.Sin(θdegree1);
                    double x = r * Math.Cos(θdegree1) * Math.Cos(φdegree1);
                    double y = r * Math.Cos(θdegree1) * Math.Sin(φdegree1);
                    pPtObs.X = pPtTar.X + x;
                    pPtObs.Y = pPtTar.Y + y;
                    pPtObs.Z = pPtTar.Z + z;
                    φdegree = φdegree1;
                    θdegree = θdegree1;
                    pCamera.Observer = pPtObs;
                    pPoint = Pt;
                    axSceneControl1.SceneGraph.RefreshViewers();
                }
                catch (Exception ex)
                {
                }

            }
        }

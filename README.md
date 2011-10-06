What is DataGridBundle?
-----

datagrid for Symfony2 highly inspired by Zfdatagrid and Magento Grid but not compatible

if you want to help or change any part according your needs im open to any idea, just create PR or open issue ticket

Compatibility
-----

Symfony - 2.0+ and will follow stable releases

Usage - controller
-----
    use Sorien\DataGridBundle\Grid\Source\Entity;
    use Sorien\DataGridBundle\Grid\Source\Document;

    class DefaultController extends Controller
    {
        public function gridAction()
        {
            // creates simple grid based on your entity (ORM)
            $grid = $this->get('grid')->setSource(new Entity('Bundle:Entity'));

            // or use Document source class for ODM
            $grid = $this->get('grid')->setSource(new Document('Bundle:Document'));

            if ($grid->isReadyForRedirect())
            {
                //data are stored, do redirect
                return new RedirectResponse($this->generateUrl('grid'));
            }
            else
            {
                // to obtain data for template you need to call prepare function
                return $this->render('YourBundle::default_index.html.twig', array('data' => $grid->prepare()));
            }
        }
    }

Usage - view
-----
    //2nd parameter is optional and defines template
    //3th parameter is optional and defines grid id, like calling $grid->setId() from controller
    {{ grid(data, 'YourBundle::own_grid_theme_template.html.twig', 'custom_grid_id') }}

your own grid theme template: you can override blocks - `grid`, `grid_titles`, `grid_filters`, `grid_rows`, `grid_pager`, `grid_actions`

    //file: YourBundle::own_grid_theme.html.twig

    {% extends 'DataGridBundle::blocks.html.twig' %}
    {% block grid %}
        extended grid!
    {% endblock %}

Usage - Document or Entity annotations
-----
    Entity and Document source uses doctrine annotations for type guessing, for better customization you can use own annotations

    use Sorien\DataGridBundle\Grid\Mapping as GRID;

    /**
     * Annotation Test Class
     *
     * @GRID\Source(columns="id, ...")
     * @GRID\Column(id="attached1", size="120", type="text") //add custom column to grid, id has to be specified
     */
    class Test
    {
        /**
         * @var integer $id
         *
         * @ORM\Column(name="id", type="integer")
         * @GRID\Column(title="my own column name", size="120", type="text", visible=true, ... )
         */
        private $id;
    }

Available types for '@GRID\Column' notation

 - id [string] - column id - default is property name, Source overrides it to field name
 - title [string] - own column name
 - size [int] - column width in pixels, default -1, -1 means auto resize
 - type [string] - column type (Date, Range, Select, Text, Boolean)
 - values [array] - options (only Select Column)
 - format [string] - format (only Date Column)
 - sortable [boolean]- turns on or off column sorting
 - filterable [boolean] - turns on or off visibility of column filter
 - source [boolean] - turns on or off column visibility for Source class
 - visible [boolean] -  turns on or off column visibility
 - primary [boolean] - sets column as primary - default is primary key form Entity/Document
 - align [string(left|right|center)] - default left,

Available types for '@GRID\Source' notation

 - columns [string] order of columns in grid (columns are separated by ",")
 - filterable [bool] turns on or off visibility of all columns

Another Examples
-----
Adding custom column from controller

    $source = new Entity('Test:Test');
    $source->setCallback(Entity::EVENT_PREPARE, function ($columns, $actions){
        $columns->addColumn(new Column(array('id' => 'callbacks', 'size' => '54', 'sortable' => false, 'filterable' => false, 'source' => false)));
        $actions->addAction('Delete', 'YourProject\YourBundle\Controller\YourControllerClass::yourDeleteMethod');
    });

Modify returned items from source

    $source->setCallback(Entity::EVENT_PREPARE_QUERY, function ($query) {
            $query->setMaxResults(1);
    });

Configure limits and first page shown, have to be done before setting source to grid

    $grid->setLimits(array(3 => '3', 6 => '6', 9 => '9'));


Many to One association support with `.` notation (just ORM)

    /**
     * @ORM\ManyToOne(targetEntity="Vendors")
     * @ORM\JoinColumn(name="vendor", referencedColumnName="id")
     *
     * @GRID\Column(id="vendor.name")
     */
    private $vendor;

Custom cell rendering inside template defined as 2nd argument in twig function `grid`

    {% block grid_column_yourcolumnid_cell %}
    <span style="color:#f00">My row id is: {{ row.getPrimaryFieldValue() }}</span>
    {% endblock %}

Custom filter rendering inside template defined as 2nd argument in twig function `grid`

    {% block grid_column_yourcolumnname_filter %}
    <span style="color:#f00">My custom filter</span>
    {% endblock %}

Working preview
-----
<img src="http://vortex-portal.com/datagrid/grid2.png" alt="Screenshot" />

you can find assets from preview on wiki 

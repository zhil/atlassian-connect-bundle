# Atlassian Connect Bundle

### Installation
##### Step 1: Download the Bundle
In `composer.json`:

    "require": {
        // ...
        "thecatontheflat/atlassian-connect-bundle": "dev-master"
    }
    
##### Step 2: Enable the Bundle
    // app/AppKernel.php
    class AppKernel extends Kernel
    {
        public function registerBundles()
        {
            $bundles = array(
                // ...
                new AtlassianConnectBundle\AtlassianConnectBundle()
            );
        }
    }

##### Step 3: Configure the Bundle

Bundle configuration includes has two main nodes - `prod` and `dev`. When requesting descriptor - this configuration is converted to JSON. Whatever you specify under `dev` node will override same option from `prod`.

Sample configuration in `config.yml`

    atlassian_connect:
        token_lifetime: 86400
        prod:
            key: 'your-addon-key'
            name: 'Your Add-On Name'
            description: 'Your Add-On Description'
            vendor:
                name: 'Your Vendor Name'
                url: 'https://marketplace.atlassian.com/vendors/1211528'
            baseUrl: 'https://your-production-domain.com/'
            lifecycle:
                installed: '/handshake'
            scopes: ['READ', 'WRITE']
            modules:
                jiraIssueTabPanels:
                    -
                        key: 'your-addon-key-tab'
                        url: '/protected/list?issue=${issue.key}'
                        weight: 100
                        name:
                            value: 'Tab Name'

        dev:
          baseUrl: 'http://localhost:8888'


##### Step 4: Configure Security

To configure security part - use the following configuration in your `security.yml`

    security:
        providers:
            jwt_user_provider:
                id: jwt_user_provider
    
        firewalls:
            jwt_secured_area:
                pattern: "^/protected"
                stateless: true
                simple_preauth:
                    authenticator: jwt_authenticator
                
##### Step 5: Include Routes

Add the following to your `app/config/routing.yml`

    ac:
        resource: "@AtlassianConnectBundle/Resources/config/routing.yml"


##### Step 6 (Optional): Configure License Check

To perform a license check for a certain route - specify the `requires_license` options in your `routing.yml`

    some_route:
        path: ...
        defaults: ...
        options:
            requires_license: true
                

##### Step 7: Update Database

    app/console doctrine:schema:update --force


# Usage Examples

### Signed Request

In your **protected** controller action you can make a signed request to JIRA instance:

    $request = new JWTRequest($this->getUser());
    $json = $request->get('/rest/api/2/issue/KEY-XXX');

### White listening licences

You could white-list any lisence by editing related row in table tenant and setting field is_white_listed to 1.
If you will also set white_listed_until - you will be able to set white-list expiration

### Dev environment

In dev environment Tenant with id=1 would be used automatically

### Custom tenant entity

You could add new entity AppBundle/Entity/JiraTenant like

    <?php
    namespace AppBundle\Entity;
    
    use Doctrine\ORM\Mapping as ORM;
    use AtlassianConnectBundle\Entity\Tenant as BaseTenant;
     
    /**
     * JiraTenant
     *
     * @ORM\Entity()
     */
    class JiraTenant extends BaseTenant
    {
        /**
         * @ORM\Column(type="string", nullable=true)
         */
        protected $cusomProperty;
    
        /**
         * @return mixed
         */
        public function getCusomProperty()
        {
            return $this->cusomProperty;
        }
    
        /**
         * @param mixed $cusomProperty
         * @return $this
         */
        public function setCusomProperty($cusomProperty)
        {
            $this->cusomProperty = $cusomProperty;
            return $this;
        }
    }
    
And override default one by setting parameter
    
    atlassian_connect_tenant_entity_class: AppBundle:JiraTenant

Since [Class Table Inheritance](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/inheritance-mapping.html#class-table-inheritance) used - you cant use class name Tenant and you will need to patch existing database with lowercased class name, like
 
    UPDATE tenant SET discr='jiratenant'

# Troubleshooting

### Cant start free trial of my plugin on Jira Cloud

As soon as you will create your plugin - you will be able to access plugin manifest via url

    https://yourplugindomain.com/atlassian-connect.json
    
You will be able to setup it in "Manage Addons" section of your Jira Cloud using "Upload addon" interface. But right now AtlassianConnectBundle support only "paid via Atlassian" model, so you will not be able to start your trial.

Instead of using manifest url directly - you should add **private listing** of your plugin, create token and get  manifest url like

    https://marketplace.atlassian.com/files/1.0.0-AC/artifact/descriptor/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/atlassian-connect.json?access-token=xxxxxxxx
    
If you will use that url from marketplace - your trial will be started automatically.

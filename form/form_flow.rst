FormFlow aka. Multi-Step Forms
==============================

Symfony provides support for building multi-step forms via the
``FormFlow`` system. A form flow lets you split a form into multiple
steps while keeping a single form instance, shared data, validation
groups, and navigation controls.

A form flow is implemented as a custom form type that defines:

* the ordered list of steps
* how the current step is stored
* how navigation between steps works
* how data is persisted between requests

Creating a Form Flow
--------------------

A form flow is created by extending ``AbstractFlowType`` and implementing
``buildFormFlow()``.

``buildFormFlow()``::

    use Symfony\Component\Form\Flow\AbstractFlowType;
    use Symfony\Component\Form\Flow\FormFlowBuilderInterface;
    use Symfony\Component\Form\Flow\Type\NavigatorFlowType;

    class UserSignUpType extends AbstractFlowType
    {
        public function buildFormFlow(FormFlowBuilderInterface $builder, array $options): void
        {
            $builder->addStep('personal', PersonalType::class);
            $builder->addStep('account', AccountType::class);
            $builder->addStep('confirmation', ConfirmationType::class);

            $builder->add('navigator', NavigatorFlowType::class);
        }
    }

Each step corresponds to a form type. Only the form of the current step
is rendered and processed.

Storing the Current Step
------------------------

A form flow needs to know which step is currently active. By default,
this is done via a property path on the form data.

You must configure the ``step_property_path`` option:

.. code-block:: php-attributes

    use Symfony\Component\OptionsResolver\OptionsResolver;

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => User::class,
            'step_property_path' => 'currentStep',
        ]);
    }

The property is automatically updated when the user moves between steps.

Processing a Multi-Step Form
----------------------------

To use a form flow in a controller, create the form, handle the request,
and check the flow state to determine what to do next::

    use App\Entity\User;
    use App\Form\UserSignUpType;
    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpFoundation\Response;

    #[Route('/signup', name: 'signup')]
    public function signup(Request $request): Response
    {
        $flow = $this->createForm(UserSignUpType::class, new User());
        $flow->handleRequest($request);

        if ($flow->isSubmitted() && $flow->isValid() && $flow->isFinished()) {
            // all steps are completed, process the final data
            // $flow->getData();

            return $this->redirectToRoute('signup_success');
        }

        return $this->render('signup.html.twig', [
            'form' => $flow->getStepForm(),
        ]);
    }

After calling ``handleRequest()``, the flow knows which button was clicked
(next, previous, finish, or reset) and acts accordingly. When the form is
valid and not yet finished, ``getStepForm()`` transitions to the next step
and returns a fresh form instance ready for rendering.

When ``isFinished()`` returns ``true``, the data object (here ``$flow->getData()``)
contains all the accumulated data from every step.

Step Ordering and Priority
--------------------------

Steps are ordered by priority (higher first). If no priority is defined,
steps are ordered by insertion order.

Example:

.. code-block:: php-attributes

    $builder->addStep('first', FirstType::class, priority: 10);
    $builder->addStep('second', SecondType::class);

Skipping Steps
--------------

Steps can be conditionally skipped using a callable:

.. code-block:: php-attributes

    $builder->addStep(
        'professional',
        ProfessionalType::class,
        skip: fn (User $data) => !$data->isWorker()
    );

Skipped steps are not rendered and are ignored by navigation.

Persisting Data Between Requests
--------------------------------

Form data can be persisted between requests using a data storage.

Symfony provides several implementations:

* ``SessionDataStorage`` — the default if no explicit data storage is configured
* ``InMemoryDataStorage`` (mainly for tests)
* ``NullDataStorage``

To configure a data storage explicitly::

    use Symfony\Component\Form\Flow\DataStorage\SessionDataStorage;

    $resolver->setDefaults([
        'data_storage' => new SessionDataStorage('signup_flow', $requestStack),
    ]);

.. note::

    Session-based storage requires an active HTTP session.

Navigation Buttons
------------------

Navigation is handled via special submit buttons:

* ``PreviousFlowType``
* ``NextFlowType``
* ``FinishFlowType``
* ``ResetFlowType``

Symfony provides a default navigator::

    use Symfony\Component\Form\Flow\Type\NavigatorFlowType;

    $builder->add('navigator', NavigatorFlowType::class);

Buttons are automatically shown or hidden depending on the current step.

By default, the navigator includes previous, next and finish buttons. To also
include a reset button, use the ``with_reset`` option::

    $builder->add('navigator', NavigatorFlowType::class, [
        'with_reset' => true,
    ]);

This adds a reset button to the navigator without needing to separately add
a ``ResetFlowType`` button.

Custom Navigation
-----------------

You can define custom navigation buttons::

    use Symfony\Component\Form\Flow\Type\NextFlowType;

    $builder->add('skip', NextFlowType::class, [
        'clear_submission' => true,
        'validate' => false,
        'validation_groups' => [],
        'include_if' => ['professional'],
    ]);

Each button defines a handler that controls the flow behavior.

Validation Groups
-----------------

Validation groups are automatically scoped to the current step.

By default, the active validation groups are
``['Default', '<current_step_name>']``.

This behavior is configured by ``FormFlowType`` and allows you to define
step-specific constraints on your data object.

Finishing and Resetting the Flow
--------------------------------

When the ``finish`` button is clicked:

* the flow is marked as finished
* the data storage is cleared (if ``auto_reset`` is enabled)
* the cursor is reset to the first step

You can check completion with the ``$form->isFinished()`` method.

Rendering Step Information
--------------------------

The form view exposes step metadata:

.. code-block:: twig

    {{ form.vars.steps }}
    {{ form.vars.visible_steps }}
    {{ form.vars.cursor }}

Each step contains:

* name
* index
* position
* is_current_step
* is_skipped
* can_be_skipped

This can be used to build progress indicators or navigation menus.

Using FormFlow with Turbo
-------------------------

When `Symfony UX Turbo`_ is enabled (which is the default in applications
created with ``symfony new --webapp``), Turbo Drive intercepts form submissions
and expects the server to return specific HTTP status codes. Without proper
handling, clicking the navigation buttons (next, previous, finish) will send
an AJAX request but the form will not move to the next step.

To make FormFlow work with Turbo, the controller must return the appropriate
HTTP status codes:

* ``200`` when the form is not yet submitted (initial rendering)
* ``422`` when the form is submitted but invalid (Turbo re-renders the form)
* ``303`` when the form is submitted, valid, and moving to the next step

Here is an example of a controller that handles this::

    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpFoundation\Response;

    #[Route('/signup', name: 'signup')]
    public function signup(Request $request): Response
    {
        $flow = $this->createForm(UserSignUpType::class, new User());
        $flow->handleRequest($request);

        if ($flow->isSubmitted() && $flow->isValid() && $flow->isFinished()) {
            // process the completed data
            // $flow->getData();

            return $this->redirectToRoute('signup_success');
        }

        $response = (new Response())->setStatusCode(match (true) {
            !$flow->isSubmitted() => Response::HTTP_OK,
            !$flow->isValid() => Response::HTTP_UNPROCESSABLE_ENTITY,
            !$flow->isFinished() => Response::HTTP_SEE_OTHER,
        });

        return $this->render('signup.html.twig', [
            'form' => $flow->getStepForm(),
        ], $response);
    }

Limitations
-----------

* Nested form flows are not supported
* Only one step form is processed per request
* Steps must be explicitly defined

Summary
-------

FormFlow provides a structured, extensible, and native solution for
multi-step forms in Symfony, without relying on external bundles.

It integrates seamlessly with the Form component, validation,
HttpFoundation, and the session system while remaining fully customizable.

.. _`Symfony UX Turbo`: https://symfony.com/bundles/ux-turbo/current/index.html

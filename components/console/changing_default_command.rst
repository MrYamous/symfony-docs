Changing the Default Command
============================

The Console component will always run the ``ListCommand`` when no command name is
passed. In order to change the default command you need to pass the command
name to the ``setDefaultCommand()`` method::

    namespace Acme\Console\Command;

    use Symfony\Component\Console\Attribute\AsCommand;
    use Symfony\Component\Console\Command\Command;
    use Symfony\Component\Console\Input\InputInterface;
    use Symfony\Component\Console\Output\OutputInterface;

    #[AsCommand(name: 'hello:world')]
    class HelloWorldCommand extends Command
    {
        protected function configure(): void
        {
            $this->setDescription('Outputs "Hello World"');
        }

        protected function execute(InputInterface $input, OutputInterface $output): int
        {
            $output->writeln('Hello World');

            return Command::SUCCESS;
        }
    }

Executing the application and changing the default command::

    // application.php
    use Acme\Console\Command\HelloWorldCommand;
    use Symfony\Component\Console\Application;

    $command = new HelloWorldCommand();
    $application = new Application();
    $application->add($command);
    $application->setDefaultCommand($command->getName());
    $application->run();

Test the new default console command by running the following:

.. code-block:: terminal

    $ php application.php

This will print the following to the command line:

.. code-block:: text

    Hello World

.. warning::

    This feature has a limitation: you cannot pass any argument or option to
    the default command because they are ignored.

Learn More!
-----------

* :doc:`/components/console/single_command_tool`

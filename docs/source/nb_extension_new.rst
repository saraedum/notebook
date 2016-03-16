New functions for notebook extensions
=====================================

_get_nbextension_metadata(package)
----------------------------------
def _get_nbextension_metadata(package):
    """Get the list of nbextension paths associated with a python package.
    Returns a tuple of (the module,             [{
        'section': 'notebook',
        'src': 'mockextension',
        'dest': '_mockdestination',
        'require': '_mockdestination/index'
    }])
    Parameters
    ----------
    package : str
        Importable Python package (no dotted-notation!) exposing the
        magic-named `_jupyter_nbextension_paths` function
    """

_get_server_extension_metadata(package)
---------------------------------------
def _get_server_extension_metadata(package):
    """Load server extension metadata from a package's magic-named path.
    Returns a tuple of (
        the package as loaded
        a list of server extension specs: [
            {
                "module": "mockextension"
            }
        ]
    )
    Parameters
    ----------
    package : str
        Importable Python package (no dotted-notation!) exposing the
        magic-named `_jupyter_server_extension_paths` function
    """
	
	
I believe that the two functions above replaced::
 
* _get_nbextension_paths()
* _get_serverextension_paths()


@parente suggested fix in nbextension notebook
This should be jupyter serverextension enable --py my_fancy_module [--sys-prefix|--user] I think. 
There is no serverextension install subcommand according to the CLI.

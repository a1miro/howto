# How to Add and Maintain External Library Dependencies in C++ CMake Projects and Yocto

## C++ CMake Project

1. **Use `find_package` to Locate the Library**
	- In your `CMakeLists.txt`, try to find the library first:
	  ```cmake
	  find_package(MyLibrary QUIET)
	  if(NOT MyLibrary_FOUND)
			include(FetchContent)
			FetchContent_Declare(
				 MyLibrary
				 GIT_REPOSITORY https://github.com/example/MyLibrary.git
				 GIT_TAG        v1.2.3
			)
			FetchContent_MakeAvailable(MyLibrary)
	  endif()
	  target_link_libraries(your_target PRIVATE MyLibrary::MyLibrary)
	  ```
	- Replace `MyLibrary` and the repository URL/tag with your actual library details.

2. **Include Headers and Link**
	- Ensure you include the library headers and link the target as shown above.

3. **Maintain the Dependency**
	- Document the library version and source.
	- Update the version/tag in `FetchContent_Declare` as needed.
	- Test your project after updates.

## Yocto Project

1. ** Check the external library in Yocto **
    - Run *bitbake-layers show-recipes | tee -a recipes.list*
    - Check if the required library is already in the list, then go to step 3 
    - Otherwise continue with the step 2 

2. **Add a Recipe for the External Library**
	- Use an existing recipe from layers like `meta-openembedded` if available, or create your own in your layer:
	  ```bitbake
	  # recipes-support/mylibrary/mylibrary_1.2.3.bb
	  SUMMARY = "MyLibrary external dependency"
	  LICENSE = "MIT"
	  SRC_URI = "git://github.com/example/MyLibrary.git;branch=main"
	  S = "${WORKDIR}/git"
	  inherit cmake
	  DEPENDS = "dependency1 dependency2"
	  ```
	- List any build dependencies in `DEPENDS`.

3. **Add the Library as a Dependency in Your Project Recipe**
	- In your main project recipe (e.g., `myproject.bb`):
	  ```bitbake
	  DEPENDS += "mylibrary"
	  ```
	- This ensures Yocto builds the external library before your project.

4. **Maintain the Dependency**
	- Update the recipe when the library version changes.
	- Test builds after updates.
	- Document changes and version requirements.

---

**Tip:** Always prefer system packages when available. Use `FetchContent` only as a fallback for missing dependencies.

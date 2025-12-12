create bash shell script that takes these inputs:
	model
	spec_file
	max_iterations
	project_dir
	timeout

and then does:

check for project_dir/.autok/spec.txt
	if it does not exist, //send initializer prompt (file)//
else
	//send coding prompt (file)//

--

e.g. autok2.sh --project-dir ../auto-test --max-iterations 1 --spec ./specs/heystack.txt --timeout 600

using (as initializer prompt):
	cat ./prompts/initializer.md | kilocode --auto --timeout 600

using (as coding prompt):
	cat ./prompts/coding.md | kilocode --auto --timeout 600

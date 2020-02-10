# PR is a work in progress and shouldn't be merged
if github.pr_title.include?("WIP") || github.pr_title.include?("wip")
    warn "🚧 PR is classed as Work in Progress, this should not be merged."
end
# Warn when the PR is too big and should be splitted
if git.lines_of_code > 1000
    warn "✂️ PR is too big, consider splitting into smaller PR"
end
# Ensure a clean commits history
if git.commits.any? { |c| c.message =~ /^Merge branch '#{github.branch_for_base}'/ }
  fail "🧹 Please rebase to get rid of the merge commits in this PR. Clean your commits."
end

# check if the branch respect the pattern of meero (Angular repo style)
actual_branch_name = github.branch_for_head
if actual_branch_name.match("(refacto|chore|fix|feat|perf|hotfix)\/s[0-9].+-.*")
    message "✅ The branch name respects the pattern category/description"
else
    fail "💥 The branch must respect the pattern category/s[number]-description" unless has_ticket_airtable_label
end

#check if the commit is not empty and is well formatted by respected the pattern of Meero (Angular repo style)
first_branch_name_token = actual_branch_name.split("/").first
last_branch_name_token = actual_branch_name.split("/").last
expected_commit_name = first_branch_name_token + "(" + last_branch_name_token + ")" + ": "
if git.commits.any? { |c| c.message =~ /#{Regexp.escape(expected_commit_name)}.+/ }
    message "✅ The commit name respects the pattern category(description):[SPACE] short description"
else
    fail "💥 The commit name must respect category(description):[SPACE] short description #{expected_commit_name}"
end

# If these are all empty something has gone wrong
if git.modified_files.empty? && git.added_files.empty? && git.deleted_files.empty?
  fail "⚠️ This PR has no changes at all."
end
has_app_changes = !git.modified_files.grep(/app/).empty?
has_test_changes = !git.modified_files.grep(/appTests/).empty?
# If changes are more than 30 lines of code, tests need to be updated too
if has_app_changes && !has_test_changes && git.lines_of_code > 30
  warn "🚦 Tests were not updated, please add or update existing tests."
end

podfile_updated = !git.modified_files.grep(/Podfile/).empty?

# Leave warning, if Podfile changes
if podfile_updated
  warn "👨‍💻 The podfile was updated, don't forget to execute a pod update"
end

# Check if there are duplicate localizable strings
duplicate_localizable_strings.check_localizable_duplicates

# This lints all Swift files and leave comments in PR if there is any issue with linting
swiftlint.config_file = './app/.swiftlint.yml'
swiftlint.lint_all_files = true
swiftlint.verbose = true
swiftlint.lint_files inline_mode: true

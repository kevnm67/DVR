# frozen_string_literal: true

@ci_files = %w[travis.yml Dangerfile Gemfile Gemfile.lock podspec]

# determine if any of the files were modified
def did_modify(files_array)
  did_modify_files = false
  files_array.each do |file_name|
    next unless git.modified_files.include?(file_name) || git.deleted_files.include?(file_name)

    did_modify_files = true
    config_files = git.modified_files.select { |path| path.include? file_name }
    message "This PR changes #{github.html_link(config_files)}"
  end

  did_modify_files
end

###
### Warnings
###

warn('Changes to CI/CD files') if did_modify(@ci_files)
warn('PR is classed as Work in Progress') if github.pr_title.include? '[WIP]'

###
### Auto label
###

if github.pr_title.include? '[WIP]'
  auto_label.wip = github.pr_json['number']
else
  auto_label.remove('WIP')
end

warn('Big PR, try to keep changes smaller if you can') if git.lines_of_code > 300

has_source_changes = !git.modified_files.grep(/Sources/).empty?
has_test_changes = !git.modified_files.grep(/Tests/).empty?

if has_source_changes && !has_test_changes
  warn('Any major changes should be reflected in the Changelog.
      Please consider adding a note there and adhere to the
      [Changelog Guidelines](https://keepachangelog.com/en/1.0.0/).')
end

# Check prose for any modifications to Markdown files
markdown_files = (git.added_files.grep(%r{.*\.md/}) + git.modified_files.grep(%r{.*\.md/}))

unless markdown_files.empty?
  # Run proselint to lint and check spelling
  prose.language = 'en-us'
  prose.ignored_words = %w[Venmo DVR podfile]
  prose.ignore_acronyms = true
  prose.ignore_numbers = true
  prose.lint_files markdown_files
  prose.check_spelling markdown_files
end

# Run SwiftLint
swiftlint.verbose = true
swiftlint.config_file = './.swiftlint.yml'
swiftlint.lint_files(inline_mode: true, fail_on_error: true)

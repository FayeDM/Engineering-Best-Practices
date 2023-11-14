## Code Development Standards

### Development

For initial development of any feature, approved for work and in the To Do column, start development in the following way:
- Move the Teamwork ticket into the In Progress column.
- Create a new feature branch based off of the latest commit to main (please ensure you git checkout main; git pull prior to checkout).
  - This branch is committed and pushed daily to Github.
- When starting work, open a draft PR for this feature, convert it to a full PR once it is done.
- For reference, when going through PR, changes may be made on Github (upstream), check and pull new commits as needed.
- When the PR is ready for review, move it to the Code Review column.

### Preparing for Code Review
For any template, block, or styling tasks which are ready to move to Code Review, please do the following:
- Add a Patterns page to Prod with all relevant sections to review; add anchored headings to link to your feature as the page will get long.
- Put links to the relevant section of the Patterns page and notes (or reference link to the Design Kit/Figma if possible) to describe where you pulled the styles for reference.
- Write up instructions for testing.

## Code Review Standards

- If there are only minor code structure issues, comment inline with granular commentary
- Avoid broad scope feedback if possible
If broad feedback is necessary, this can be spun into a backlog ticket for discussion
- Provide inline comments and reviews with specific guidance on code changes/bugs
- If there are minor code changes required, but everything else in the Pull Request is ready to be approved:
  - note the requested changes and send it back to the Code Owner or use the Suggestion functionality in GitHub
  - The PR can then be Approved
  - The proposed changes should be noted in the Teamwork ticket and the ticket should be reassigned to the Code Owner for merge and deploy once the changes have been implemented
  - Once Approved, the ticket can stay in the Code Review column and be assigned to the Code Owner, tagging them with a comment, notifying them of approval

<h2 id="communication" class="anchor-heading">Effective Communication During Code Review {% include Util/link_anchor anchor="communication" %} {% include Util/top %}</h2>

This section is meant to support both engineers submitting a PR for code review and the code reviewer.

Effective communication during a code review is essential for ensuring that code quality is maintained and that the author and the reviewer benefit from the process. Here are some guidelines on how to communicate during a code review:

1. **Be Respectful and Constructive**:
   - Keep the tone respectful and professional. Avoid personal attacks or criticism of the author.

2. **Focus on the Code, Not the Author**:
   - Frame comments in terms of the code's quality and adherence to standards rather than the author's abilities or intentions.

3. **Use a Positive Approach**:
   - Start with positive feedback, pointing out what's good about the code. Highlighting strengths helps create a more constructive atmosphere.

4. **Be Specific and Clear**:
   - Be explicit in your comments, explaining why a change is needed and providing specific examples or suggestions for improvement.

5. **Prioritize Comments**:
   - Focus on the most critical issues first to avoid overwhelming the author with feedback. Reserve minor style or formatting issues for later in the review.

6. **Ask Questions**:
   - Ask questions to seek clarification when you encounter something you don’t understand. This can help avoid misunderstandings and uncover issues.

7. **Suggest Solutions**:
   - If you identify a problem, try to suggest a solution or alternative approach. Don't just point out issues; offer constructive guidance.

8. **Use Code Review Tools and Platforms**:
   - If your team uses code review tools or platforms, utilize their features for commenting, tracking changes, and discussing code. This helps maintain a record of the review process.

9. **Encourage Discussion**:
   - Encourage open and respectful discussion if there’s uncertainty or a difference of opinion. It can lead to better solutions and shared understanding.

10. **Keep Comments Contextual**:
    - Provide context for your comments. Explain why a particular change is necessary or how it fits into the broader project goals.

11. **Offer Praise and Recognition**:
    - Acknowledge the author's efforts and improvements. Positive reinforcement can motivate better code quality.

12. **Avoid Jargon and Ambiguity**:
    - Use clear and plain language in your comments. Avoid technical jargon that might not be familiar to all team members.

13. **Be Mindful of Time and Deadlines**:
    - Reviewers should be aware of time constraints and deadlines. Aim to provide timely feedback and complete the review within the agreed-upon timeframe.

14. **Consider the Impact**:
    - Think about the impact of your feedback. Will the suggested changes be easy or difficult for the author to implement? Be mindful of the trade-offs.

15. **Balance Nitpicking and Critical Feedback**:
    - While catching critical issues is essential, be mindful of not nitpicking every minor detail. Balance your feedback to focus on what truly matters.

16. **Use Emojis or Comment Tags (Optional)**:
    - Some teams use emojis (e.g., 👍 for approval or 🤔 for questions) or comment tags (e.g., TODO, FIXME) to convey feedback efficiently.

17. **Review Your Own Comments**:
    - Before submitting your review, review your comments to ensure they are clear, respectful, and actionable.

18. **Follow Up and Iterate**:
    - Review the changes after the author addresses your feedback and offer further guidance if needed. Consider having a follow-up discussion or review.

19. **Learn and Improve Together**:
    - Use code reviews as learning opportunities for both the author and the reviewer. Share knowledge, best practices, and coding standards.

20. **Accept Feedback as a Reviewer**:
    - Be open to feedback from the author. Code reviews are a two-way learning process, and both parties can benefit from constructive feedback.

Effective communication during code reviews is crucial for maintaining a positive team environment and improving code quality. Strive for a collaborative and respectful atmosphere where everyone is committed to producing the best possible code.

<h2 id="questions" class="anchor-heading">Questions for Code Review {% include Util/link_anchor anchor="questions" %} {% include Util/top %}</h2>

As a reviewer during a code review, asking thoughtful questions is essential to ensure the code is high quality, follows best practices, and meets the project's requirements. Here are some good questions to ask during a code review:

1. **Purpose and Design**:
   - What problem is this code solving?
   - Is the code consistent with the design and architecture decisions?
   - Are there any alternative approaches that could be more efficient or maintainable?

2. **Readability and Maintainability**:
   - Is the code easy to read and understand?
   - Are variable and function names descriptive and meaningful?
   - Are there any comments or documentation that could improve clarity?
   - Is the code following the Kanopi-defined naming conventions?

3. **Code Structure and Organization**:
   - Is the code organized logically and modularly?
   - Are there any code smells or anti-patterns (e.g., long methods or classes)?
   - Does the code follow the established coding standards?

4. **Error Handling and Edge Cases**:
   - How does the code handle errors and exceptions?
   - Have edge cases and boundary conditions been considered and addressed?
   - Are there potential issues related to resource leaks or unexpected failures?

5. **Testing and Validation**:
   - Are there unit tests or automated tests covering the changes?
   - Have corner cases and various input scenarios been tested?
   - Are test cases and test data provided, and do they cover the code changes adequately?

6. **Performance and Efficiency**:
   - Is the code efficient in terms of time and space complexity?
   - Are there any potential bottlenecks or performance issues?
   - Have best practices for database queries and network requests been followed?

7. **Security**:
   - Does the code handle user input and data securely?
   - Are there any security vulnerabilities, such as SQL injection, XSS, or data exposure?
   - Are authentication and authorization mechanisms correctly implemented?

8. **Code Duplications and Reusability**:
   - Are there any duplicated code segments that could be refactored into reusable functions or classes?
   - Can parts of the code be generalized for broader use?

9. **Documentation and Comments**:
   - Is there sufficient inline documentation to explain complex logic or algorithms?
   - Are code comments clear and informative, focusing on "why" rather than "what" the code is doing?

10. **Consistency and Coding Standards**:
    - Does the code adhere to the team's coding standards and style guide?
    - Are there any inconsistencies or deviations from established practices?

11. **Dependencies and Third-party Libraries**:
    - Are there any dependencies or third-party libraries that should be reviewed separately?
    - Are these dependencies up-to-date and secure?

12. **Compatibility and Cross-Browser/Platform Support**:
    - Has the code been tested across browsers or platforms for web or cross-platform applications?
    - Are there any compatibility issues that need addressing?

13. **Scalability and Future Considerations**:
    - Will this code scale well as the application grows?
    - Are there any considerations for future maintenance or enhancements?

14. **Version Control and Git Practices**:
    - Are commits granular and logically organized?
    - Are there any code conflicts or issues with branching and merging?

15. **User Experience (UX)**:
    - Does the code impact the user experience, and if so, is it consistent with the design and usability requirements?

16. **Performance Profiling and Profiler Output**:
    - If applicable, have performance profiling tools been used, and what do they reveal about the code's performance?

17. **Feedback and Clarifications**:
    - Are there any areas where the author would like specific feedback or has questions for the reviewer?

Remember that the goal of asking these questions is not to criticize the author but to ensure that the code is high-quality, adheres to best practices, and meets the project's requirements. Effective communication between the reviewer and the author is essential to address issues and foster collaboration.

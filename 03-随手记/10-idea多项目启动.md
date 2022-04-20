# idea多项目以service方式显示





1. 找到 .idea文件夹

2. 找到 workspace.xml 文件

3. 找到 component name="RunDashboard"这个节点整个替换掉  

   - **注：如果没有直接添加**

   ```xml
   <component name="RunDashboard">
     <option name="configurationTypes">
      <set>
       <option value="SpringBootApplicationConfigurationType" />
      </set>
     </option>
     <option name="ruleStates">
      <list>
       <RuleState>
        <option name="name" value="ConfigurationTypeDashboardGroupingRule" />
       </RuleState>
       <RuleState>
        <option name="name" value="StatusDashboardGroupingRule" />
       </RuleState>
      </list>
     </option>
    </component>
   ```

   
---
title:  Springboot自定义异常处理
date: 2016-05-31 20:39:12
tags:
- SpringBoot
categories:
- Spring
toc: true
---

### 背景 
Springboot 默认把异常的处理集中到一个ModelAndView中了，但项目的实际过程中，这样做，并不能满足我们的要求。具体的自定义异常的处理，参看以下

### 前提
1. [Springboot 默认的application properties](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#common-application-properties%20Springboot%E9%BB%98%E8%AE%A4%E7%9A%84%20application%20properties)
2. [Spring Boot异常处理详解](http://www.cnblogs.com/xinzhao/p/4934247.html%20Spring%20Boot%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E8%AF%A6%E8%A7%A3)
<!-- more -->
### 具体实现
如果仔细看完Spring boot的异常处理详解，并且研究过源码后，我觉得具体的实现可以不用看了。。。
重写定义错误页面的url，默认只有一个/error
```java
@Bean
public EmbeddedServletContainerCustomizer containerCustomizer(){
	return new EmbeddedServletContainerCustomizer(){
		@Override
		public void customize(ConfigurableEmbeddedServletContainer container) {
			container.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/error/404"));
			container.addErrorPages(new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error/500"));
			container.addErrorPages(new ErrorPage(java.lang.Throwable.class,"/error/500"));
		}
	};
}
```
重写通过实现ErrorController，重写BasicErrorController的功能实现
```java
**
 * 重写BasicErrorController,主要负责系统的异常页面的处理以及错误信息的显示
 * @see org.springframework.boot.autoconfigure.web.BasicErrorController
 * @see org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration
 *
 * @author Jonathan
 * @version 2016/5/31 11:22
 * @since JDK 7.0+
 */
@Controller
@RequestMapping(value = "error")
@EnableConfigurationProperties({ServerProperties.class})
public class ExceptionController implements ErrorController {

	private ErrorAttributes errorAttributes;

	@Autowired
	private ServerProperties serverProperties;


	/**
	 * 初始化ExceptionController
	 * @param errorAttributes
	 */
	@Autowired
	public ExceptionController(ErrorAttributes errorAttributes) {
		Assert.notNull(errorAttributes, "ErrorAttributes must not be null");
		this.errorAttributes = errorAttributes;
	}


	/**
	 * 定义404的ModelAndView
	 * @param request
	 * @param response
	 * @return
	 */
	@RequestMapping(produces = "text/html",value = "404")
	public ModelAndView errorHtml404(HttpServletRequest request,
	                              HttpServletResponse response) {
		response.setStatus(getStatus(request).value());
		Map<String, Object> model = getErrorAttributes(request,
				isIncludeStackTrace(request, MediaType.TEXT_HTML));
		return new ModelAndView("error/404", model);
	}

	/**
	 * 定义404的JSON数据
	 * @param request
	 * @return
	 */
	@RequestMapping(value = "404")
	@ResponseBody
	public ResponseEntity<Map<String, Object>> error404(HttpServletRequest request) {
		Map<String, Object> body = getErrorAttributes(request,
				isIncludeStackTrace(request, MediaType.TEXT_HTML));
		HttpStatus status = getStatus(request);
		return new ResponseEntity<Map<String, Object>>(body, status);
	}

	/**
	 * 定义500的ModelAndView
	 * @param request
	 * @param response
	 * @return
	 */
	@RequestMapping(produces = "text/html",value = "500")
	public ModelAndView errorHtml500(HttpServletRequest request,
	                              HttpServletResponse response) {
		response.setStatus(getStatus(request).value());
		Map<String, Object> model = getErrorAttributes(request,
				isIncludeStackTrace(request, MediaType.TEXT_HTML));
		return new ModelAndView("error/500", model);
	}


	/**
	 * 定义500的错误JSON信息
	 * @param request
	 * @return
	 */
	@RequestMapping(value = "500")
	@ResponseBody
	public ResponseEntity<Map<String, Object>> error500(HttpServletRequest request) {
		Map<String, Object> body = getErrorAttributes(request,
				isIncludeStackTrace(request, MediaType.TEXT_HTML));
		HttpStatus status = getStatus(request);
		return new ResponseEntity<Map<String, Object>>(body, status);
	}


	/**
	 * Determine if the stacktrace attribute should be included.
	 * @param request the source request
	 * @param produces the media type produced (or {@code MediaType.ALL})
	 * @return if the stacktrace attribute should be included
	 */
	protected boolean isIncludeStackTrace(HttpServletRequest request,
	                                      MediaType produces) {
		ErrorProperties.IncludeStacktrace include = this.serverProperties.getError().getIncludeStacktrace();
		if (include == ErrorProperties.IncludeStacktrace.ALWAYS) {
			return true;
		}
		if (include == ErrorProperties.IncludeStacktrace.ON_TRACE_PARAM) {
			return getTraceParameter(request);
		}
		return false;
	}


	/**
	 * 获取错误的信息
	 * @param request
	 * @param includeStackTrace
	 * @return
	 */
	private Map<String, Object> getErrorAttributes(HttpServletRequest request,
	                                               boolean includeStackTrace) {
		RequestAttributes requestAttributes = new ServletRequestAttributes(request);
		return this.errorAttributes.getErrorAttributes(requestAttributes,
				includeStackTrace);
	}

	/**
	 * 是否包含trace
	 * @param request
	 * @return
	 */
	private boolean getTraceParameter(HttpServletRequest request) {
		String parameter = request.getParameter("trace");
		if (parameter == null) {
			return false;
		}
		return !"false".equals(parameter.toLowerCase());
	}

	/**
	 * 获取错误编码
	 * @param request
	 * @return
	 */
	private HttpStatus getStatus(HttpServletRequest request) {
		Integer statusCode = (Integer) request
				.getAttribute("javax.servlet.error.status_code");
		if (statusCode == null) {
			return HttpStatus.INTERNAL_SERVER_ERROR;
		}
		try {
			return HttpStatus.valueOf(statusCode);
		}
		catch (Exception ex) {
			return HttpStatus.INTERNAL_SERVER_ERROR;
		}
	}

	/**
	 * 实现错误路径,暂时无用
	 * @see ExceptionMvcAutoConfiguration#containerCustomizer()
	 * @return
	 */
	@Override
	public String getErrorPath() {
		return "";
	}

}
```

### 总结
第一步，通过定义containerCustomizer,重写定义了异常处理对应的视图。目前定义了404和500，可以继续扩展。
第二步，重写BasicErrorController,当然可以直接定义一个普通的controller类，直接实现第一步定义的视图的方法。重写的目的是重用ErrorAttributes。这样在页面，直接可以获取到status,message,exception,trace等内容。
如果仅仅是把异常处理的视图作为静态页面，不需要看到异常信息内容的话，直接第一步后，再定义error/404，error/500等静态视图即可。

ErrorController根据Accept头的内容，输出不同格式的错误响应。比如针对浏览器的请求生成html页面，针对其它请求生成json格式的返回

以上两步的操作，比网上流传的更能实现自定义化。
